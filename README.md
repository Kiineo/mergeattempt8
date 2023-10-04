# Jenkins Project README

This README provides an overview of the Jenkins CI/CD Implementation project and its key components. The project focuses on setting up Jenkins for continuous integration and delivery (CI/CD) to automate the build, testing, and deployment processes.

## Project Components

### EC2 Instance for Jenkins Server

- An EC2 instance is provisioned and managed to host the Jenkins Server.
- This enables continuous integration and delivery for the project.

### Amazon Elastic Container Registry (ECR)

- An Amazon Elastic Container Registry (ECR) is established to efficiently store deployable application images.
- ECR ensures seamless deployments and version control of containerized applications.

### Jenkins Pipeline

The Jenkins Pipeline is the heart of our CI/CD implementation. It automates various stages of the development lifecycle, ensuring consistent and reliable builds, tests, and deployments.

#### Event 1: Creation of Jenkins EC2 Instance

- An EC2 instance is provisioned and managed to host the Jenkins Server.
- This enables continuous integration and delivery for the project.

#### Event 2: Fetching the Code from GitHub

- The project code is fetched from the GitHub repository using the following configuration:

    ```groovy
    stage('Fetch code') {
        steps{
            git branch: 'vp-rem', url:'https://github.com/devopshydclub/vprofile-repo.git'
        }  
    }
    ```

#### Event 3: Unit Testing (Maven)

- Unit tests are executed using Maven with the following command:

    ```groovy
    stage('Test'){
        steps {
            sh 'mvn test'
        }
    }
    ```

#### Event 4: Checkstyle (Maven)

- Code style and quality checks are performed using Checkstyle with the following command:

    ```groovy
    stage('Checkstyle Analysis'){
        steps {
            sh 'mvn checkstyle:checkstyle'
        }
    }
    ```

#### Event 5: Code Analysis (SonarQube)

- Code analysis is performed using SonarQube with the following configuration:

    ```groovy
    stage('Sonar Analysis') {
        environment {
            scannerHome = tool 'sonar4.7'
        }
        steps {
           withSonarQubeEnv('sonar') {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
               -Dsonar.projectName=vprofile \
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

#### Event 6: Docker Build

- Docker images for the application are built and managed using Docker Pipeline and AWS ECR.

#### Event 7: Pushing the Code to Amazon ECR

- The Docker images are pushed to Amazon Elastic Container Registry (ECR) for efficient storage and version control.

#### Event 8: Quality Gate

- A quality gate ensures the build meets predefined quality criteria. It waits for the quality gate to pass before proceeding.

#### Event 9: Upload Artifact

- Artifacts, such as the application WAR file, are uploaded to Nexus Repository Manager for further distribution.

### Dockerization

- The application is orchestrated for Dockerization.
- Docker Pipeline and AWS ECR are used for image creation and management.
