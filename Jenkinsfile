pipeline {
    agent any

    directory="build"
    pattern="index.html"

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
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Test') {
            steps {
                sh '''
                    find "$directory" -type f | grep "$pattern"
                    npm test
                '''
            }
        }
    }
}
