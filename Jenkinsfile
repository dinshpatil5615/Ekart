pipeline {
    agent any
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    tools {
        maven 'maven3'
        jdk 'jdk-17'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/dinshpatil5615/Ekart.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh """${env.SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=EKART \
                        -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=target/classes"""
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk-17', maven: 'maven3', traceability: true) {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t dineshpatil0908/ekart:latest docker/Dockerfile .'
            }
        }

        stage('Push Image to Hub') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'DOCKERHUB_PWD')]) {
                    sh '''
                        echo $DOCKERHUB_PWD | docker login -u dineshpatil0908 --password-stdin
                        docker push dineshpatil0908/ekart:latest
                        docker logout
                    '''
                }
            }
        }

        stage('EKS and Kubectl Configuration') {
            steps {
                script {
                    sh 'aws eks update-kubeconfig --region us-east-1 --name project-cluster'
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                script {
                    sh 'kubectl apply -f deploymentservice.yml'
                }
            }
        }
    }
}
