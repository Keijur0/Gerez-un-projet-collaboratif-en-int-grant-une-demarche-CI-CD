# Documentation des GitHub Actions pour le projet Bobapp

## 1. Introduction
Le repository GitHub "Bobapp" contient deux workflows de CI/CD, l'un pour le backend et le second pour le frontend de l'application. Ces workflows permettent d'automatiser la compilation, les tests, la génération du rapport de couverture de code par les tests, l'analyse de la qualité de code, ainsi que la génération et le déploiement des images Docker.  
Ce document a pour objectif de détailler chaque étape des GitHub Actions, de proposer des KPIs et de rapport les premières métriques collectées après l'exécution des pipelines. Le document inclura également l'analyse des retours d'utilisateurs pertinents, de manière à identifier les améliorations nécessaires à mettre en place en priorité.  

## 2. Liens vers le projet SonarCloud et le repository DockerHub

### [SonarCloud](https://hub.docker.com/repositories/keijur0)  

### [DockerHub](https://hub.docker.com/repositories/keijur0)    

## 3. Etapes des GitHub Actions

## Pipeline Backend CI/CD
Fichier: `backend-ci-cd.yml`  
Emplacement: `.github/workflows`

### Déclenchement du workflow
- **Push** sur les chemins `back/**` et `.github/workflows/**` dans la branche main.
- **Pull request** sur les chemins `back/**` et `.github/workflows/**` dans la branche main, pour les événements `opened`, `synchronize` et `reopened`.

### Jobs
1. **`build_test_and_analyze`**

**Objectifs**: Builder le backend, exécuter les tests, générer le rapport de couverture et analyser la qualité du code avec SonarCloud.

### Etapes:
1.1. **Checkout code**
``` yaml
- name: Checkout code
  uses: actions/checkout@v4
```
**Objectif**: Récupération du code source du repository.
___

1.2. **Set up JDK 17**
``` yaml
- name: Set up JDK 17
  uses: actions/setup-java@v4
  with:
    java-version: '17'
    distribution: 'adopt'
```
**Objectif**: Installer JDK 17, qui sera nécessaire pour la compilation du code et l'exécution des tests.
___

1.3. **Cache Maven packages**
``` yaml
- name: Cache Maven packages
  uses: actions/cache@v4
  with:
    path: ~/.m2
    key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
    restore-keys: ${{ runner.os }}-m2
```
**Objectif**: Mise en cache des dépendances Maven pour accélérer le build.
___

1.4. **Build with Maven**
``` yaml
- name: Build with Maven
  run: mvn clean verify -X
```
**Objectif**: Compilation du code et exécution des tests avec Maven.
___

1.5. **Generate JaCoCo report**
``` yaml
- name: Generate JaCoCo report
  run: mvn jacoco:report
```
**Objectif**: Génération du rapport de couverture avec JaCoCo.
___

1.6. **Upload JaCoCo report**
``` yaml
- name: Upload JaCoCo report
  uses: actions/upload-artifact@v4
  with:
    name: jacoco-report
    path: back/target/site/jacoco
```
**Objectif**: Enregistrement du rapport de couverture et le rendre accessible sur GitHub.
___

1.7. **Cache SonarQube packages**
``` yaml
- name: Cache SonarQube packages
  uses: actions/cache@v4
  with:
    path: ~/.sonar/cache
    key: ${{ runner.os }}-sonar
    restore-keys: ${{ runner.os }}-sonar
```
**Objectif**: Mise en cache des paquets SonarQube pour optimiser le temps d'éxécution des analyses.
___

1.8. **Analyze with SonarCloud**
``` yaml
- name: Analyze with SonarCloud
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
  run: mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=keijur0-bobapp-back -Dsonar.projectName='keijur0_bobapp_back'
```
**Objectif**: Analyse du code avec SonarCloud pour vérifier sa qualité et détecter d'éventuels bugs ou vulnéraibilités.
___

2. **`build_and_push_docker_image`**

Nécessite l'exécution avec succès de `build_test_and_analyze`
**Objectif**: Builder et déployer une image Docker du backend sur DockerHub.

2.1. **Checkout code**
``` yaml
- name: Checkout code
  uses: actions/checkout@v4
```
**Objectif**: Récupération du code source pour la construction de l'image Docker.
___

2.2. **Set up Docker Buildx**
``` yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3
```
**Objectif**: Configuration de l'outil Docker Buildx pour des builds multi-plateformes.
___

2.3. **Cache Docker layers**
``` yaml
- name: Cache Docker layers
  uses: actions/cache@v4
  with:
    path: /tmp/.buildx-cache
    key: ${{ runner.os }}-buildx-${{ github.sha }}
    restore-keys: |
      ${{ runner.os }}-buildx-
```
**Objectif**: Mise en cache des couches Docker pour accélérer les builds.
___

2.4. **Login to DockerHub**
``` yaml
- name: Login to DockerHub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_PASSWORD }}
```
**Objectif**: Authentification sur DockerHub pour permettre le déploiement de l'image Docker.
___

2.5. **Build Docker image**
``` yaml
- name: Build Docker image
  run: |
    docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/bobapp-backend:latest .
```
**Objectif**: Construire l'image Docker du backend.
___

2.6. **Push Docker image**
``` yaml
- name: Push Docker image
  run: |
    docker push ${{ secrets.DOCKERHUB_USERNAME }}/bobapp-backend:latest
```
**Objectif**: Déploiement de l'image Docker du backend sur Dockerhub.
___

## Pipeline Frontend CI/CD
Fichier: `frontend-ci-cd.yml`  
Emplacement: `.github/workflows`

### Déclenchement du workflow
- Push sur les chemins `front/**` et `.github/workflows/**` dans la branche main.
- Pull request sur les chemins `front/**` et `.github/workflows/**` dans la branche main, pour les événements `opened`, `synchronize` et `reopened`.

### Jobs
1. **`build_test_and_analyze`**

**Objectifs**: Builder le frontend, exécuter les tests, générer le rapport de couverture et analyser la qualité du code avec SonarCloud.

### Etapes:
1.1. **Checkout code**
``` yaml
- name: Checkout code
  uses: actions/checkout@v4
```
**Objectif**: Récupération du code source du repository.
___

1.2. **Set up Node.js**
``` yaml
- name: Set up Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '18'
```
**Objectif**: Installation de Node.js, qui sera nécessaire pour le build du frontend.
___

1.3. **Cache Node.js modules**
``` yaml
- name: Cache Node.js modules
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```
**Objectif**: Mise en cache des modules Node.js pour accélérer les installations.
___

1.4. **Install dependencies**
``` yaml
- name: Install dependencies
  run: npm install
```
**Objectif**: Installation des dépendances nécessaires au build et à l'exécution des tests.
___

1.5. **Run tests with coverage**
``` yaml
- name: Run tests with coverage
  run: npm run test -- --code-coverage --browsers=ChromeHeadless --watch=false
```
**Objectif**: Exécution des tests et génération d'un rapport de couverture.
___

1.6. **Upload coverage report**
``` yaml
- name: Upload coverage report
  uses: actions/upload-artifact@v4
  with:
    name: coverage-report
    path: front/coverage/
```
**Objectif**: Enregistrement du rapport de couverture et le rendre accessible sur GitHub.
___

1.7. **Analyze with SonarCloud**
``` yaml
- name: Analyze with SonarCloud
  uses: SonarSource/sonarcloud-github-action@master
  with:
    projectBaseDir: ./front
  env:
    GITHUB_TOKEN: ${{ secrets.GH_TOKEN }}
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```
**Objectif**: Analyse du code du frontend avec SonarCloud pour vérifier sa qualité et détecter des bugs ou vulnérabilités.
___


2. **`build_and_push_docker_image`**

Nécessite l'exécution avec succès de `build_test_and_analyze`  
**Objectif**: Builder et déployer une image Docker du frontend sur DockerHub.

2.1. **Checkout code**
``` yaml
- name: Checkout code
  uses: actions/checkout@v4
```
**Objectif**: Récupérer le code source pour la construction de l'image Docker.
___

2.2. **Set up Docker Buildx**
``` yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3
```
**Objectif**: Configuration de l'outil Docker Buildx pour des builds multi-plateformes.
___

2.3. **Cache Docker layers**
``` yaml
- name: Cache Docker layers
  uses: actions/cache@v4
  with:
    path: /tmp/.buildx-cache
    key: ${{ runner.os }}-buildx-${{ github.sha }}
    restore-keys: |
      ${{ runner.os }}-buildx-
```
**Objectif**: Mise en cache des couches Docker pour accélérer les builds.
___

2.4. **Login to DockerHub**
``` yaml
- name: Login to DockerHub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_PASSWORD }}
```
**Objectif**: Authentification sur DockerHub pour permettre le déploiement de l'image Docker.
___

2.5. **Build Docker image**
``` yaml
- name: Build Docker image
  run: |
    docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/bobapp-frontend:latest .
```
**Objectif**: Construire l'image Docker du frontend.
___

2.6. **Push Docker image**
``` yaml
- name: Push Docker image
  run: |
    docker push ${{ secrets.DOCKERHUB_USERNAME }}/bobapp-frontend:latest
```
**Objectif**: Déploiement de l'image Docker du frontend sur DockerHub.
___

## 4. Proposition de KPIs
Pour garantir l'amélioration continue de la qualité de BobApp, je propose 4 KPIs critiques. Ces KPIs permettront de suivre et d'évaluer l'efficacité des tests, la sécurité, la maintenabilité et la fiabilité du code au fur et à mesure de son développement.  

1. **Couverture de code par les tests (Code coverage): 80%**

Ce KPI consiste à mesurer le pourcentage de code source couvert par les tests automatisés. Une couverture élevée permettra de garantir que la majorité du code a été testée, ce qui permet de réduire les risques de bugs en production.

Le seuil minimum que je recommande pour ce KPI est de **80%**.  

Ce pourcentage permet d'assurer que la majorité du code a été testée, mais il laisse également une certaine marge de flexibilité car certaines parties du code sont parfois difficile à tester.  
Ce seuil est aligné sur les standards de qualité Sonar Way proposés par SonarQube. Il offre un bon équilibre entre l'effort déployé pour l'implémentation des tests et la détection des problèmes potentiels.

2. **Evaluation de la fiabilité du code (Reliability Rating): A**

Ce KPI consiste à évaluer la fiabilité du code en fonction des bugs détectés. Ils sont classés en fonction de leur sévérité par SonarQube, grâce à une note qui va de A (la meilleure note) à E (la pire note).

Le seuil minimum que je recommande pour ce KPI est **A**.

Cette note permet d'indiquer qu'aucun nouveau bug critique n'a été introduit dans le code ajouté. Cela est essentiel pour maintenir la stabilité de l'application et éviter des régressions qui pourraient nuire à l'expérience utilisateur.

3. **Evaluation de la sécurité du code (Security Rating): A**

Ce KPI consiste à mesurer la sécurité du code en fonction des vulnérabilités détectées par SonarQube. Comme pour la fiabilité, la sécurité est notée de A à E, A étant la meilleure note.

Le seuil minimum que je recommande pour KPI est **A**.

Ce score permet de confirmer qu'aucune vulnérabilité critique n'a été introduite dans le code. Assurer un niveau de sécurité élevé est crucial pour protéger les données des utilisateurs et éviter les failles qui pourraient entraîner des dommages significatifs à l'intégrité de l'application et à la confiance des utilisateurs.

4. **Evaluation de la maintenabilité du code (Maintainability Rating): A**

Ce KPI consiste à évaluer la maintenabilité du code enfonction de la dette technique identifiée par SonarQube. La dette technique inclut des aspects tels que la complexité cyclomatique, les duplications de code et la structure générale du code. Les notes vont de A à E, où A est la meilleure note.

Le seuil minimum que je recommande pour KPI est **A**.

Ce seuil permet d'indiquer que le code est bien structuré, facile à comprendre et à maintenir. Cela est essentiel pour permettre des évolutions rapides et éviter des coûts élevés de maintenance sur le long terme.

## 5. Analyse des métriques

## 6. Analyse des retours utilisateurs

Les retours des utilisateurs de BobApp ont fourni des indications précieuses sur les aspects de l'application qui nécessitent une amélioration immédiate.  
Voici un résumé des principaux retours:

```
★☆☆☆☆  
Je mets une étoile car je nep eux pas en mettre zéro ! Impossible de poster une suggestion de blague, le bouton tourne
et fait planter mon navigateur !
```
**Problème:** L'utilisateur mentionne un bug avec un bouton de suggestion de blague, mais cette fonctionnalité n'existe pas dans l'application.  
**Action:** Malgré l'incohérence du commentaire, il soulève tout de même une évolution possible. Il serait envisageable d'introduire cette fonctionnalité pour répondre aux attentes des utilisateurs.
___
```
★★☆☆☆  
#BobApp j'ai remonté un bug sur le post de vidéo il y a deux semaines et il est encore présent ! Les devs vous faites quoi ????
```
**Problème:** L'utilisateur signale un bug sur le post de vidéo, mais cette fonctionnalité n'existe pas dans l'application.  
**Action:** De même que pour le commentaire précédent, il soulève une fonctionnalité qui pourrait être ajoutée dans les futures versions de l'application pour répondre à la demande des utilisateurs.
___
```
★☆☆☆☆  
Ca fait une semaine que je ne reçois plus rien, j'ai envoyé un email il y a 5 jours mais toujours pas de nouvelles...
```
**Problème:** L'utilisateur ne reçoit pas de réponse à leur demande d'assistance par email.
**Action:**  Améliorer la réactivité du support en proposant un suivi des demandes sous forme de GitHub Issues pour une gestion plus efficace des problèmes.  
___
```
★★☆☆☆
J'ai supprimé ce site de mes favoris ce matin, dommage, vraiment dommage.
```
**Problème:** L'utilisateur est déçu et a supprimé l'application de ses favoris. Il n'a pas précisé la raison, mais il est probable que cela vienne de bugs récurrents dont il a fait l'expérience denièrement.  
**Action:** Augmenter la qualité globale de l'application en corrigeant les bugs existants et en introduisant de nouvelles fonctionnalités attractives pour regagner la confiance des utilisateurs.


## 7. Actions prioritaires

Sur la base des métriques fournies par les couvertures de code, les analyses de SonarCloud et les retours utilisateurs, voici les actions prioritaires à mener pour améliorer BobApp:

1. Résolution des bugs existants

- Corriger les bugs signalés pour améliorer l'expérience utilisateur.
- S'assurer que toutes les fonctionnalités actuelles fonctionne correctement, et en particulier celles mentionnées par les utilisateurs.

2. Amélioration des réponses aux support

- Augmenter la réactivité du support en traitant les demandes via GitHub Issues, ce qui facilitera le suivi et la résolution des problèmes remontés.

3. Augmentation de la couverture de code

- Backend: Ajouter des tests unitaires et d'intégration pour atteindre le seuil minimum recommandé (80%).  
- Frontend: Améliorer la couverture des fonctions en testant les fonctionnalités critiques de l'application.

4. Maintien du niveau de sécurité

- Continuer à prêter une attention particulière au score de sécurité pour conserver la note "A" au fur et à mesure de l'évolution de l'application.

5. Amélioration de la maintenabilité du code

- Backend: Simplifier le code et améliorer la structure globale du code pour obtenir la note "A" en maintenabilité.

6. Prise en compte des retours utilisateurs

- Introduire de nouvelles fonctionnalités, telles que la suggestion de blague et le post de vidéo, pour retenir les utilisateurs et en attirer de nouveaux.  
- Assurer une communication régulière et transparente avec les utilisateurs pour maintenir leur engagement et répondre à leurs attentes.

Ces actions permettront d'améliorer la qualité, la fiabilité, la sécurité et la maintenabilité de l'application BobApp, tout en répondant efficacement aux attentes des utilisateurs.
