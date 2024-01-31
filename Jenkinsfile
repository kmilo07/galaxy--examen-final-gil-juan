pipeline {
    agent any
        environment {
            DOCKER_CREDS = credentials('docker-credentials')
        }
        stages {
            stage('Build') {
                agent {
                    docker { image 'maven:3.6.3-openjdk-11-slim' }
                }
                steps {
                    sh 'mvn -B verify install'
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, onlyIfSuccessful: true
                }
                post {
                    success {
                        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, onlyIfSuccessful: true
                    }
                }
            }
        }
}