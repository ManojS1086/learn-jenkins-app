pipeline {
    agent any

    stages {
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm --version
                npm ci
                npm run build         
                '''
            }
        }

        stage('Test') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm test       
                '''
            }

            post{
                always {
                    junit 'jest-results/junit.xml'
                }
            }
        }

        stage('E2E Tests') {
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm install serve
                node_modules/.bin/serve -s build &
                sleep 10
                npx playwright test --reporter=html       
                '''
            }
            post {
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright-report', reportTitles: '', useWrapperFileDirectly: true])
                }                
            }
        }
        stage('AWS-prod-Deploy') {
            agent{
                docker{
                    image 'amazon/aws-cli:latest'
                    args "--entrypoint ''"
                    reuseNode true
                }
            }
            steps {
                sh '''
                aws --version
                aws s3 sync build s3://jenkins-bucket-22-06-2025-10-49
                '''
            }
        }
    }
}