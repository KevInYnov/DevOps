
# Introduction

Ce compte rendu porte sur le module DevOps.

Il nous a été demandé de réaliser une prise de notes pour chaque cours effectué et d'alimenter un dépôt GitLab avec ces notes. Cependant, j'ai remarqué que cela ne donnerait pas un rendu de qualité pour mon cas, en raison des absences et de l'oubli de prises de notes sur certains modules.

J'ai donc pris la décision de me concentrer sur la prise de notes concernant la réalisation des objectifs demandés, qui sont les suivants :

- Créer, alimenter et documenter un dépôt GitLab
- Avoir un fichier GitLab CI qui s'exécute
- Créer un Dockerfile compilable
- Réaliser le build de l'image Docker et pousser l'image sur le Container Registry du projet
- Ajouter une analyse SAST GitLab à notre CI
- Réaliser un scan SonarScanner et pousser le résultat automatiquement sur SonarQube
- Réaliser un scan de sécurité sur notre image Docker
- Déployer le projet sur un serveur

Cela permettra de réaliser un compte rendu de qualité tout en abordant les sujets traités lors du module DevOps.

Vous trouverez néanmoins les prises de notes effectuées durant les cours dans le fichier `RETEX.md`, qui se trouve dans le dossier : `Rapport -> Retex`.

# Environnement

Afin de réaliser nos objectifs, j'ai mis en place un environnement de travail sur une machine virtuelle, bien que nous aurions pu le faire avec de la conteneurisation via Docker.

J'ai choisi de travailler sur un système d'exploitation Parrot (une distribution Linux) pour ce projet, et ce, pour plusieurs raisons. 
Tout d'abord, étant dans une spécialisation en cybersécurité, j'apprends à utiliser les outils que je serai amené à maîtriser dans ma future vocation, et rien de mieux que la pratique pour cela. 
De plus, ces systèmes incluent déjà les logiciels dont nous avons besoin (Docker, Git, Trivy).

# Réalisation du projet

## Première étape

La première étape consiste à récupérer le [repo Gitlab fournit](https://gitlab.com/Glandalf/web-project-sample)

Pour ce faire, utilisez les commandes suivantes :  
`git clone https://gitlab.com/Glandalf/web-project-sample`

![[1.png]]

Vous verrez que le projet a bien été cloné et que le dossier `web-project-sample` est présent.

Maintenant, nous allons créer un compte GitLab et un dépôt où déposer notre travail au fil du temps. 
Une fois la création du dépôt terminée, nous devons pointer vers ce dépôt :  
`git remote add origin https://gitlab.com/kevinynov1/devops`  
`git branch -M main` (! vérifiez que la branche main n'est pas protégée !)  
`git push -uf origin main`

Une fois cela réalisé, vous devriez voir le projet téléchargé dans votre dépôt.

![[2.png]]

Nous avons donc réussi à configurer correctement notre dépôt. Nous devons maintenant ajouter l'utilisateur `@Glandalf` en tant que `maintainer` afin que notre super intervenant puisse accéder au dépôt.

Pour ajouter un utilisateur avec le rôle `maintainer` : `Projet -> Members -> Invite Member`

![[3.png]]

![[4.png]]

L'utilisateur Glandalf peut maintenant accéder à notre projet. 
Le déploiement du `web-project-sample` a été abordé dans les prises de notes effectuées en cours, c'est pourquoi nous passerons ce chapitre dans ce compte rendu.

## Deuxième étape 

Maintenant que notre projet est créé sur GitLab, nous allons créer un GitLab CI qui aura pour objectif de déployer un conteneur Docker et d'effectuer une série de tests.

GitLab CI est un outil intégré dans GitLab qui automatise les processus de développement, test et déploiement du code. Il permet d'exécuter des tâches répétitives comme les tests, la compilation et l'analyse de sécurité à chaque modification du code.

Pour ce faire, nous allons créer un fichier `.gitlab-ci.yml` qui affichera "Bonjour" dans un premier temps.

![[5.png]]

Explication :

- **stages** : définit les étapes du pipeline (ici, une seule étape `test`).
- **echo_job** : un job qui sera exécuté dans l'étape `test`.
- **script** : contient les commandes à exécuter dans ce job, ici simplement un `echo "Hello"`.

Pour tester que notre CI fonctionne correctement, nous allons pousser le fichier sur notre dépôt GitLab.

![[6.png]]

Puis, pour tester que tout fonctionne comme prévu, allez dans l'onglet `Build -> Pipelines`.

![[7.png]]

Nous pouvons voir que deux des trois jobs affichent une erreur. 
En effet, vous devrez vérifier votre compte avec un numéro de téléphone pour pouvoir accéder aux fonctionnalités liées au CI de GitLab.

Maintenant, pour vérifier que l'affichage du "Hello" fonctionne correctement, allez dans l'onglet `Build -> Jobs` et sélectionnez le dernier job affiché.

![[8.png]]

Nous verrons bien l'affichage du "Hello", ce qui nous permet de conclure que le CI de GitLab interprète correctement l'exécution des étapes configurées dans le fichier `.gitlab-ci.yml`.
## Troisième étape

En lisant le terminal du job, nous remarquons que le système d'exploitation par défaut du CI GitLab est `ruby:3.1`, or imaginons que, pour les besoins de notre projet, nous avons besoin d'un système d'exploitation `Debian` avec `Python3` installé.

Pour ce faire, nous devrons tout d'abord créer l'image Docker en question (`Debian` + `Python3`).

Tout d'abord, il nous faudra créer un fichier `Dockerfile` sans extension.

![[9.png]]

Explication :

- `FROM debian:bullseye` : Utilise **Debian Bullseye** (une version stable de Debian) comme OS.    
- `ENV DEBIAN_FRONTEND=noninteractive` : Empêche les choix pendant l'installation.
- `RUN apt-get update` : Met à jour la liste des paquets disponibles.
- `apt-get install -y python3 python3-pip` : Installe Python 3 et pip.
- `CMD ["python3", "--version"]` : Affiche la version de Python installée.

Avant de pousser ce fichier dans notre dépôt, nous allons tester notre image en local pour vérifier que tout fonctionne correctement.

Pour ce faire, exécutez : `docker build -t debian-python .` (le `.` indique que le Dockerfile se trouve dans le répertoire actuel).

![[10.png]]

Nous pourrons afficher l'image créée avec la commande `docker images`.

![[11.png]]

Maintenant, pour vérifier que notre image Docker fonctionne correctement, nous allons lancer un conteneur temporaire avec notre image. Le résultat attendu est l'affichage de la version de Python3.  
Pour ce faire, utilisez : `docker run --rm debian-python3`.

![[12.png]]

Maintenant que notre `Dockerfile` est prêt, il nous suffit de le pousser dans notre dépôt GitLab. Pour ce faire :  
`git add Dockerfile`  
`git commit -m "add dockerfile debian + python"`  
`git push`

![[13.png]]

Maintenant que nous avons notre `Dockerfile` sur notre dépôt, nous pouvons passer à l'étape suivante.

## Quatrième étape

Nous avons donc notre `Dockerfile` prêt. 
Nous allons maintenant modifier notre fichier `.gitlab-ci.yml` afin que le CI de GitLab déploie notre système d'exploitation `Debian` avec `Python3` installé pour effectuer nos phases de test.

Mais avant cela, il est important de comprendre le service `Docker Container Registry`.

Le `Docker Container Registry` est un service qui permet de stocker et gérer des images Docker.

Le but principal est de centraliser et partager les images Docker, mais il sert également aux `Pipelines`. 
Les images peuvent être automatiquement poussées vers le registre après un build réussi.

Pour cela, nous utiliserons celui fourni par GitLab, le `GitLab Container Registry`, un registre privé pour stocker des images Docker dans GitLab (exclusif au projet).

Nous allons maintenant modifier notre fichier `.gitlab-ci.yml` comme suit :

![[14.png]]

Explication :

- `image: docker:latest` : Utilise une image Docker officielle.
- `docker:dind` : Active Docker-in-Docker pour pouvoir construire les images.
- `variables: DOCKER_DRIVER: overlay2` : Variables d'environnement.
- `before_script: docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" "$CI_REGISTRY"` : Avant chaque étape, nous effectuons un login au Docker Registry.
- `docker build -t "$CI_REGISTRY_IMAGE:latest" .` : Construit l'image Docker à partir du Dockerfile.
- `docker push "$CI_REGISTRY_IMAGE:latest"` : Pousse l'image sur le Container Registry du projet.

Une fois le fichier de configuration du CI poussé, nous pouvons vérifier son fonctionnement de la même manière que pour le premier job créé :  
`Build -> Jobs`

![[15.png]]

Le job a bien été exécuté avec succès, et les variables d'environnement du CI ont bien fonctionné. 
Notez que la configuration de `Python3` apparaît clairement dans les logs, ce qui confirme que c'est bien notre image.

Après l'exécution de ce pipeline, l'image sera disponible dans le **Container Registry** de notre projet GitLab.

Pour le vérifier, allez dans le service suivant : `Deploy -> Container Registry`

![[16.png]]

Nous avons donc réalisé le build de l'image Docker et poussé l'image sur le Container Registry du projet.

## Cinquième étape

Maintenant, nous allons ajouter une analyse SAST GitLab à notre CI.

L’analyse **SAST (Static Application Security Testing)** dans GitLab est un outil de sécurité automatisé qui permet de détecter les failles de sécurité dans le code source de ton application, sans l’exécuter.

Pour ce faire, nous commencerons par créer un script Python vulnérable, qui sera notre fichier `malware.py` (nom discret avant tout).

![[17.png]]

**Pourquoi ce script est vulnérable ?**  

Parce qu'il utilise `subprocess` avec `shell=True` sans filtrer les entrées utilisateur. Cela peut permettre une **injection de commande**, qui sera détectée par un outil comme `Bandit`, utilisé par GitLab SAST pour Python.

Nous allons maintenant ajouter l'analyse **SAST** à notre pipeline. Heureusement, GitLab fournit une intégration prête à l'emploi via ses templates CI/CD de sécurité.

Pour ce faire, modifions le fichier `.gitlab-ci.yml` et ajoutons un job `stats` :

![[18.png]]

Explication :

- `image: registry.gitlab.com/security-products/semgrep:6` : Utilise une image Docker contenant Semgrep, un outil open-source de scan statique.
- `apk add docker-cli` : Ajoute `docker` à l’image.
- `semgrep --config=auto .` : Lance Semgrep avec une configuration automatique sur le répertoire courant.

Maintenant, il nous suffit de pousser les fichiers dans notre dépôt :  
`git add .` ou `git add malware.py .gitlab-ci.yml`  
`git commit -m "Add: SAST"`  
`git push`

Nous constaterons que notre `Pipelines` a bien effectué deux jobs.

![[19.png]]

Maintenant, allons consulter notre job `stats` pour vérifier ce que dit l'audit de sécurité.

![[20.png]]

Nous constatons que notre `SAST` a bien identifié un problème dans le fichier `malware.py`. Il avertit que l’utilisation de `subprocess.check_output()` avec `shell=True` peut être dangereuse.

Nous constatons également qu'il a relevé une erreur dans le `Dockerfile`, en effet, celui-ci utilise un terminal `CMD` en `root` car nous n'avons pas spécifié d'utilisateur, ce qui peut être exploité par un attaquant pour exécuter un reverse shell en Python directement sur l'utilisateur `root`.