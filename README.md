# Weather App CI/CD Deployment

This repository provides a complete CI/CD pipeline setup for deploying a **Weather App** using **GitLab CI/CD** and **Jenkins**. The pipelines automate building, testing, pushing, and deploying the application using **Docker**.

## 🚀 Features
- **Automated CI/CD**: Supports both **GitLab CI/CD** and **Jenkins Pipelines**.
- **Docker-based deployment**: Ensures portability and easy deployment.
- **Automated testing**: Runs Jest tests and generates test reports.
- **Push to Docker Hub**: Publishes built images to **Docker Hub**.
- **Automated deployment**: Deploys the containerized application.

---

## 🛠 GitLab CI/CD Pipeline

The GitLab pipeline consists of the following stages:
1. **Prepare**: Cleans up and sets up the workspace.
2. **Checkout**: Clones the repository.
3. **Authenticate**: Logs into Docker Hub.
4. **Build**: Builds the Docker image.
5. **Test**: Installs dependencies and runs unit tests.
6. **Push**: Pushes the Docker image to Docker Hub.
7. **Deploy**: Runs the containerized app.

### GitLab `.gitlab-ci.yml`
```yaml
stages:
  - prepare
  - checkout
  - authenticate
  - build
  - test
  - push
  - deploy

variables:
  WORKSPACE: "/home/gitlab-runner/app"
  DOCKER_IMAGE: "piyushh69/weather-app"   #use your docker username 
  DOCKER_TAG: "latest"
  TEST_RESULTS_DIR: "$WORKSPACE/Weather-app/test-results"

default:
  tags:
    - gitlab-runner-redhat-new

before_script:
  - docker --version
  - node --version
  - npm --version

checkout_code:
  stage: checkout
  script:
    - git clone https://github.com/Piyushshrii/javascript-projects.git $WORKSPACE

build_image:
  stage: build
  script:
    - cd $WORKSPACE
    - docker build -t $DOCKER_IMAGE:$DOCKER_TAG -f Weather-app/Dockerfile Weather-app/

deploy:
  stage: deploy
  script:
    - docker rm -f weather-app-container || true
    - docker run -d -p 80:80 --name weather-app-container $DOCKER_IMAGE:$DOCKER_TAG
```

---

## 🏗 Jenkins Pipeline

This Jenkins pipeline follows a similar structure:
1. **Checkout Code**: Clones the repository.
2. **Login to Docker Hub**: Authenticates with Docker Hub.
3. **Build Docker Image**: Builds the application image.
4. **Install Dependencies**: Installs required packages.
5. **Run Unit Tests**: Executes Jest tests and stores results.
6. **Push to Docker Hub**: Publishes the image.
7. **Deploy Container**: Runs the latest container.

### Jenkins `Jenkinsfile`
```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "piyushh69/weather-app"   #use your docker username 
        DOCKER_TAG = "latest"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jenkins-server-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$DOCKER_TAG -f Weather-app/Dockerfile Weather-app/'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'cd Weather-app && npm test || true'
            }
            post {
                always {
                    junit 'Weather-app/test-results/results.xml'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh 'docker push $DOCKER_IMAGE:$DOCKER_TAG'
            }
        }

        stage('Deploy Container') {
            steps {
                sh 'docker rm -f weather-app-container || true'
                sh 'docker run -d -p 80:80 --name weather-app-container $DOCKER_IMAGE:$DOCKER_TAG'
            }
        }
    }
}
```

---

## 📌 Prerequisites
- **GitLab Runner** installed and configured.
- **Jenkins** installed with required plugins.
- **Docker** installed on the runner/agent.
- **Node.js & NPM** installed for running tests.
- **Credentials** for Docker Hub stored securely in GitLab/Jenkins.

---

## 🛠 How to Use
### **GitLab CI/CD**
1. Add `.gitlab-ci.yml` to the root of your repository create a new project in gitlabs.
2. Ensure GitLab Runner is properly set up replace your configured runner name in the script yaml file.
3. Setup DOCKER_USER AND DOCKER_PASS environmnet variable from project CICD setting
4. Dont forget to docker to usergroup
5. Push changes to trigger the pipeline.

### **Jenkins**
1. Add `Jenkinsfile` to the repository.
2. Configure a Jenkins pipeline job pointing to your repo
3. Setup DOCKER_USER AND DOCKER_PASS environmnet variable to global credentials and set id as 'jenkins-server-id'
4. Dont forget to docker & jenkins to usergroup
5. Run the job to execute the pipeline.

---

## 🎯 Conclusion
Voilaaa! Access the application on the public ip of your server with port 80
This setup provides an **automated, containerized CI/CD solution** for deploying your Weather App using GitLab CI/CD or Jenkins. Choose the one that fits your workflow! 🚀

**Feel free to fork, modify, and contribute!** 😊

