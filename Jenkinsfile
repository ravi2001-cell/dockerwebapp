pipeline {
    agent any 
    
    tools {
        maven 'mymaven'
    }
    stages {
        stage('code') {
            steps {
               git 'https://github.com/ravi2001-cell/dockerwebapp.git'
            }
        }
        stage('code scan') {
            steps {
               withSonarQubeEnv('sonar') {
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=myproject -Djacoco.skip=true -DskipTests"
                }
            }
        }
        stage('code build') {
            steps {
               sh 'mvn clean package -DskipTests'
            }
        }
        stage('artifact') {
            steps {
               // Fixed: Removed 'http://' and the trailing slash from nexusUrl
               nexusArtifactUploader artifacts: [[artifactId: 'vprofile1.101', classifier: '', file: 'target/vprofile1.101-v1.10.war', type: 'war']], credentialsId: 'nexus', groupId: 'com.visualpathit', nexusUrl: '35.175.228.36:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'myrepo', version: 'v1.10'
            }
        } // FIXED: Added these missing closing brackets for the artifact stage
        stage('code build using docker') {
            steps {
               sh 'docker build -t rkdocker1800/auto:app Docker-app'
               sh 'docker build -t rkdocker1800/auto:db Docker-db'
            }
        }
        stage('image scan') {
            steps {
              sh 'trivy image rkdocker1800/auto:app'
              sh 'trivy image rkdocker1800/auto:db'
            }
        }
        stage('docker push') {
            steps {
                script { 
                     withDockerRegistry(credentialsId: 'dockerpassword') {
                         sh 'docker push rkdocker1800/auto:app'
                         sh 'docker push rkdocker1800/auto:db'
                     }
                }
            }
        }
        stage('deployment') {
            steps {
               sh 'docker compose down --remove-orphans'
               sh 'docker compose up -d'
            }
        }
    }
}
