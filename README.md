# DevSecOps Project - Dotnet Monitoring Application

This project demonstrates the implementation of DevSecOps practices using a Dotnet Monitoring application. The following tools and technologies are employed for CI/CD, security scanning, and deployment automation.

![image](https://github.com/user-attachments/assets/22462044-16b5-4d6a-8c09-8c82684572fd)
![image](https://github.com/user-attachments/assets/1665bd04-383b-4978-828b-f83e271be48c)
![image](https://github.com/user-attachments/assets/3ad78973-1468-403a-ad18-093a91f1f263)


## Tools & Technologies Used

- **CI/CD Tool**: Jenkins
- **Security Tools**: 
  - Trivy (Filesystem & Image Scanning)
  - SonarQube (Code Quality & Security Analysis)
  - OWASP Dependency Check (Vulnerability Detection)
- **Deployment**: Docker, Kubernetes (Minikube)
- **Version Control System**: Git and GitHub

## CI/CD Pipeline with Jenkins

This project utilizes a Jenkins pipeline to automate the build, test, and deployment processes. The pipeline integrates with security and monitoring tools to ensure a robust and secure deployment workflow.

![image](https://github.com/user-attachments/assets/073a1a4f-dae4-47d9-959d-b5ddc148a0e1)

### Key Pipeline Stages:
1. **Build & Test**: Automated builds using Jenkins for the Dotnet Monitoring Application.
2. **Code Quality & Security Analysis**: SonarQube is integrated to check code quality, detect bugs, and identify security vulnerabilities.
3. **Filesystem & Image Scanning**: Trivy performs both filesystem and image scans to detect security issues in dependencies and container images.
4. **Vulnerability Detection**: OWASP Dependency Check ensures no known vulnerabilities exist in the application's dependencies.
5. **Docker Build & Push**: The application is containerized using Docker. Jenkins automates the Docker image build and pushes it to a Docker registry (e.g., Docker Hub) for future deployments.
6. **Deployment to Kubernetes**: The application is deployed to a Kubernetes cluster running on Minikube. The service is accessed using the `kubectl port-forward` command to forward the necessary ports for accessing the application.

## Security Tools Integration

### SonarQube Analysis
SonarQube is integrated into the Jenkins pipeline to provide continuous inspection of code quality and security. It checks for code bugs, code smells, and security vulnerabilities.

![image](https://github.com/user-attachments/assets/49be5f5b-e5f1-4533-aefb-e39741e28b90)

### Trivy File System & Image Scans
Trivy is used to perform filesystem scans to detect security issues in the project's dependencies. Additionally, image scanning ensures that Docker images are free from vulnerabilities before deployment.

![image](https://github.com/user-attachments/assets/9a0e2c8c-679b-482d-8a32-adb8cf66e8f7)

### OWASP Dependency Check
OWASP Dependency Check is used to analyze the project dependencies for known vulnerabilities. It integrates into the pipeline to halt deployment if critical vulnerabilities are found.

![image](https://github.com/user-attachments/assets/41e0e2b7-ef8c-42f4-8466-ad2ce278061d)

## Deployment with Docker and Kubernetes

### Docker Build & Push
The application is containerized using Docker, and the image is built and pushed to a Docker registry for future deployments.

![image](https://github.com/user-attachments/assets/fd5c1a78-401b-4fc7-9360-b2153619bd15)

### Kubernetes Deployment (Minikube)
The application is deployed in a Kubernetes cluster running on Minikube. Minikube provides a local Kubernetes environment for testing and development purposes.

The service is accessed using the `kubectl port-forward` command to forward the necessary ports for accessing the application.

![image](https://github.com/user-attachments/assets/ef28567f-4714-44fb-be55-b5f0a818bd1f)

```Bash
pipeline {
    agent any
    
    environment{
        SCANNER_HOME = tool 'sonar-scanner'
        TRIVY_HOME="<PATH>"
        DOCKER="<PATH>"
        KUBECTL_HOME=="<PATH>"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: '371fd4b1-3e74-4a3e-8898-06c624b0ab2d', url: 'https://github.com/TanmayRao7/DotNet-Application.git'
                sh 'ls -lhtr'
            }
        }
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectName=dotnetapp -Dsonar.projectKey=dotnetapp'
                }
            }
        }
        stage('Trivy FS') {
            steps {
                sh '${TRIVY_HOME}/trivy fs . > fs-scan.txt '
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'checker'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Docker Build') {
            steps {
                sh '${DOCKER}/docker build -t tanmayrao7/dotnetapp:latest .  --file  build/Dockerfile'
            }
        }
        stage('Docker Push') {
            steps {
                sh '${DOCKER}/docker push tanmayrao7/dotnetapp:latest'
            }
        }
        stage('Trivy Image') {
            steps {
                sh '${TRIVY_HOME}/trivy image tanmayrao7/dotnetapp:latest > image-scan.txt '
            }
        }
        stage('Deploy To K8s') {
            steps {
                sh '${KUBECTL_HOME}/kubectl apply -f deployment.yaml'
                sh '${KUBECTL_HOME}/kubectl get all'
            }
        }
    }
}

---

