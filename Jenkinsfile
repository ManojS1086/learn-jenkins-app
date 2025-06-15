pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                ls -la
            }
        }

        stage('Test'){
            agent{
                docker{
                    image 'node:18-alpine'
                }
            }
            steps{
                sh 'cd build && grep "index.html" .'
                npm test
            }
        }
    }
}