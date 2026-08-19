pipeline {
    agent any

    tools{
        nodejs "nodejs-26.7.0"
    }
    stages {
        stage('VM Node Version') {
            steps {
                sh '''
                    node -v
                    npm -v
                '''
            }
        }
    }
}
