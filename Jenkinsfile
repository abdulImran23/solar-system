pipeline {
    agent any

    tools{
        nodejs "nodejs-26.7.0"
    }
    stages {
        stage('Installing Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }
    }
}
