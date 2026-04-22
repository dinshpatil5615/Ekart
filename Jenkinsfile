pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        // NVD_API_KEY = credentials('nvd-api-key')  // Jenkins secret text credential
    }

    tools {
        maven 'maven3'
        jdk 'jdk-17'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/dinshpatil5615/Ekart.git'
            }
        }

        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('unit tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh "${env.SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=EKART \
                        -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=target/classes"
                }
            }
        }

        // stage('OWASP Dependency Check') {
        //     steps {
        //         withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
        //             dependencyCheck additionalArguments: "--nvdApiKey ${NVD_API_KEY} --scan target/ --format HTML --format XML --project EKART --out .", 
        //                             odcInstallation: 'DC'
        //         }
        //     }
        //     post {
        //         always {
        //             dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //             archiveArtifacts artifacts: '**/dependency-check-report.html',
        //                              allowEmptyArchive: true
        //         }
        //     }
        // }

        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }

        stage('deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk-17', maven: 'maven3', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        

        stage('Push Image to Hub') {
            steps{
        	    withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'DOCKERHUB_PWD')]) {
                    sh '''
                        echo $DOCKERHUB_PWD | docker login -u dineshpatil0908 --password-stdin
                        docker push dineshpatil0908/ekart:latest
                        docker logout
        }
        
        stage('EKS and Kubectl configuration'){
            steps{
                script{
                    sh 'aws eks update-kubeconfig --region us-east-1 --name project-cluster'
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                script{
                    sh 'kubectl apply -f deploymentservice.yml'
                }
            }
        }
    }

}
