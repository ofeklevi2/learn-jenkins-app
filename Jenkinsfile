pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:10-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    nom ci
                    npm run build
                    ls la
                '''
            }
        }
    }
}
