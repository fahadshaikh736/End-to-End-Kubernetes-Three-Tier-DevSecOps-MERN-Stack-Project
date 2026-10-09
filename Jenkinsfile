pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node18'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Code Quality - Backend') {
            steps {
                dir('Application-Code/backend') {
                    withSonarQubeEnv('sonar-server') {
                        sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=backend -Dsonar.projectName=backend -Dsonar.sources=."
                    }
                }
            }
        }

        stage('Code Quality - Frontend') {
            steps {
                dir('Application-Code/frontend') {
                    withSonarQubeEnv('sonar-server') {
                        sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=frontend -Dsonar.projectName=frontend -Dsonar.sources=."
                    }
                }
            }
        }

        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs --severity HIGH,CRITICAL --exit-code 0 --format table -o trivy-fs-report.txt Application-Code'
                archiveArtifacts artifacts: 'trivy-fs-report.txt', allowEmptyArchive: true
            }
        }
    }
}