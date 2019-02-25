pipeline {
    agent {label 'jenkins-slave-mvn'}

    stages {
        stage('Build') {
            steps {
                echo 'Building'
                sh 'mvn compile'
                sh 'sleep 300'
            }
        }
        stage('Unit Testing') {
            steps {
                sh 'mvn test'
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