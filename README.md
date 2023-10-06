# Jenkins CI/CD Pipeline README

This README provides an overview of the Jenkins CI/CD pipeline used for automating various stages of your project's development lifecycle.

## Project Overview
![JenkinsProject](https://github.com/Kiineo/Jenkins-VProfile-Project/assets/103956412/a7d16aff-cb04-4f92-9ad9-2cfb9a89a672)

## Pipeline Components

### 1. Creation of the Jenkins EC2 Instance

- The Jenkins CI/CD pipeline runs on an EC2 instance.
- This instance is provisioned and managed to host the Jenkins Server.

### 2. Fetching the Code from GitHub

- The project's source code is fetched from the GitHub repository using the following configuration:

    ```groovy
    stage('Fetch code'){
      steps {
        git branch: 'docker', url: 'https://github.com/Kiineo/vprofile-project.git'
      }
    }
    ```

### 3. Unit Testing (Maven)

- The pipeline performs unit tests using Maven with the following command:

    ```groovy
    stage('Test'){
      steps {
        sh 'mvn test'
      }
    }
    ```

### 4. Checkstyle (Maven)

- Code style and quality checks are performed using Checkstyle with the following command:

    ```groovy
    stage ('CODE ANALYSIS WITH CHECKSTYLE'){
        steps {
            sh 'mvn checkstyle:checkstyle'
        }
        post {
            success {
                echo 'Generated Analysis Result'
            }
        }
    }
    ```

### 5. Code Analysis (SonarQube)

- Code analysis is performed using SonarQube with the following configuration:

    ```groovy
    stage('build && SonarQube analysis') {
        environment {
            scannerHome = tool 'sonar4.7'
        }
        steps {
            withSonarQubeEnv('sonar') {
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                -Dsonar.projectName=vprofile-repo \
                -Dsonar.projectVersion=1.0 \
                -Dsonar.sources=src/ \
                -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                -Dsonar.junit.reportsPath=target/surefire-reports/ \
                -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }
        }
    }
    ```

### 6. Docker Build

- The pipeline builds a Docker image for your application using the Dockerfile located in `./Docker-files/app/multistage/`.

    ```groovy
    stage('Build App Image') {
        steps {
            script {
                dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
            }
        }
    }
    ```

### 7. Pushing the Code to Amazon ECR

- The Docker image is pushed to Amazon Elastic Container Registry (ECR) for efficient storage and version control.

    ```groovy
    stage('Upload App Image') {
        steps{
            script {
                docker.withRegistry( vprofileRegistry, registryCredential ) {
                    dockerImage.push("$BUILD_NUMBER")
                    dockerImage.push('latest')
                }
            }
        }
    }
    ```

This Jenkins CI/CD pipeline automates the project's build, testing, code analysis, Docker image creation, and deployment to ECR, ensuring a streamlined development and deployment process.
