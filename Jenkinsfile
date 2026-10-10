pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node18'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        AWS_REGION   = 'ap-south-1'
        ACCOUNT_ID   = sh(returnStdout: true, script: 'aws sts get-caller-identity --query Account --output text').trim()
        REGISTRY     = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
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

        stage('Build Docker Images') {
            steps {
                sh "docker build -t ${REGISTRY}/three-tier-backend:${BUILD_NUMBER} Application-Code/backend"
                sh "docker build -t ${REGISTRY}/three-tier-frontend:${BUILD_NUMBER} Application-Code/frontend"
            }
        }

        stage('Push to ECR') {
            steps {
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${REGISTRY}"
                sh "docker push ${REGISTRY}/three-tier-backend:${BUILD_NUMBER}"
                sh "docker push ${REGISTRY}/three-tier-frontend:${BUILD_NUMBER}"
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --severity HIGH,CRITICAL --exit-code 0 --format table -o trivy-backend-image.txt ${REGISTRY}/three-tier-backend:${BUILD_NUMBER}"
                sh "trivy image --severity HIGH,CRITICAL --exit-code 0 --format table -o trivy-frontend-image.txt ${REGISTRY}/three-tier-frontend:${BUILD_NUMBER}"
                archiveArtifacts artifacts: 'trivy-*-image.txt', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f || true'
        }
    }
}