pipeline {
    agent any
    
    options {
        disableConcurrentBuilds()
    }
    
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
               sh 'mkdir -p Docker-app/target'
               sh 'cp target/*.war Docker-app/target/'
            }
        }
        stage('artifact') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def pomGroupId = pom.groupId ?: pom.parent.groupId
                    nexusArtifactUploader(artifacts: [[artifactId: pom.artifactId, classifier: '', file: "target/${pom.artifactId}-${pom.version}.war", type: 'war']], credentialsId: 'nexus', groupId: pomGroupId, nexusUrl: '35.175.228.36:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'myrepo', version: pom.version)
                }
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
                    withDockerRegistry(credentialsId: 'docker'){
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
