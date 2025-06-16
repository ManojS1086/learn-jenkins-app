pipeline {
    agent any

    environment{
        NETLIFY_AUTH_TOKEN = credentials('app-token')
        NETLIFY_SITE_ID = '122620dd-df02-44be-80f0-430d4ee80183'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                ls -la
                '''
            }
        }

        stage(parallelblock){
            parallel{
                stage('E2E'){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 10
                        npx playwright test --reporter=html
                        '''
                    }

                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }

                stage('Test'){
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                        ls -la
                        test -f build/index.html
                        npm test
                        '''
                    }
                    post{
                        always{
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
            }
        }

        stage('preprod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm install netlify-cli@20.1.1
                node_modules/.bin/netlify --version
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }

        stage('approval'){
            steps{
                timeout(time: 1, unit: 'HOURS') {
                input message: 'are we ready to deploy to prod?', ok: 'Yes, we can deploy to production'
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm install netlify-cli@20.1.1
                node_modules/.bin/netlify --version
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage('E2E'){
            environment{
                CI_ENVIRONMENT_URL= 'https://lovely-brigadeiros-a1174a.netlify.app'
            }
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps{
                sh '''
                npx playwright test --reporter=html
                '''
            }

            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright HTML prod Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}