pipeline {
    agent any

    stages {
        stage('first-stage') {
            steps {
                cleanWs()
                echo 'first pipeline in code...'
                sh '''
                    echo this is our first pipeline
                '''
            }
        }
    }
}
