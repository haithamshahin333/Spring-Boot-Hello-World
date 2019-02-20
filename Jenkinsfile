pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Sleeping..'
                sh 'sleep 300'
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}