pipeline {
    agent {label 'jenkins-slave-mvn'}

    stages {
        stage('Build') {
            steps {
                echo 'printing env'
                sh 'printenv'
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