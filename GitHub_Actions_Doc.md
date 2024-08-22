# Documentation des GitHub Actions pour le projet Bobapp

## 1. Introduction
Le repository GitHub "Bobapp" contient deux workflows de CI/CD, l'un pour le backend et le second pour le frontend de l'application. Ces workflows permettent d'automatiser la compilation, les tests, la génération du rapport de couverture de code par les tests, l'analyse de la qualité de code, ainsi que la génération et le déploiement des images Docker.
Ce document a pour objectif de détailler chaque étape des GitHub Actions, de proposer des KPIs et de rapport les premières métriques collectées après l'exécution des pipelines. Le document inclura également l'analyse des retours d'utilisateurs pertinents, de manière à identifier les améliorations nécessaires à mettre en place en priorité.

## 2. Etapes des GitHub Actions

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

## 3. Proposition de KPIs

## 4. Analyse des métriques

## 5. Analyse des retours utilisateurs
