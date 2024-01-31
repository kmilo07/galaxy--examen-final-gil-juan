pipeline {
    agent any
    environment {
        DOCKER_CREDS = credentials('docker-credentials')
    }
    stages {
        stage('Checkout repository') {
            steps {
                cleanWs()
                checkout scm
            }
        }
        stage('Build') {
            agent {
                docker { image 'maven:3.6.3-openjdk-11-slim' }
            }
            steps {
                sh 'mvn verify'
                sh 'mvn clean package'
                sh 'ls -l target/'
            }
            post {
                success {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, onlyIfSuccessful: true
                }
            }
        }
        stage('SonarQube') {
            agent {
                docker {
                image 'maven:3.6.3-openjdk-11-slim'
                }
            } 
            steps {
                script{
                    def scannerHome = tool 'scanner-default'
                    withSonarQubeEnv('sonar-server') {
                        sh "${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=finalmaven01 \
                        -Dsonar.projectName=finalmaven01 \
                        -Dsonar.sources=src/main \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.language=java \
                        -Dsonar.tests=src/test \
                        -Dsonar.junit.reportsPath=target/surefire-reports \
                        -Dsonar.surefire.reportsPath=target/surefire-reports \
                        -Dsonar.jacoco.reportPath=target/jacoco.exec \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.java.coveragePlugin=jacoco \
                        -Dsonar.coverage.jacoco.xmlReportPaths=target/jacoco.xml \
                        -Dsonar.exclusions=**/*IT.java,**/*TEST.java,**/*Test.java,**/src/it**,**/src/test**,**/gradle/wrapper** \
                        -Dsonar.java.libraries=target/*.jar"
                    }
                }
            }
        }
        stage('Build Docker') {
            steps {
                copyArtifacts filter: 'target/labmaven-*.jar',
                                fingerprintArtifacts: true,
                                projectName: '${JOB_NAME}',
                                flatten: true,
                                selector: specific('${BUILD_NUMBER}'),
                                target: 'target';
                sh 'docker --version'
                sh 'docker-compose --version'
                sh 'docker-compose build'
            }
        }
    }
}
