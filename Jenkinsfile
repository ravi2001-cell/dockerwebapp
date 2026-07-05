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
               sh 'rm -rf Docker-app/target'
               sh 'mvn clean package -DskipTests'
               sh 'cp -r target Docker-app'
            }
        }
        stage('artifact') {
            steps {
               nexusArtifactUploader artifacts: [[artifactId: 'vprofile', classifier: '', file: 'target/vprofile-v2.war', type: 'war']], credentialsId: 'nexuspassword', groupId: 'com.visualpathit', nexusUrl: '35.175.228.36:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'myrepo', version: 'v2'
            }
        }
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
