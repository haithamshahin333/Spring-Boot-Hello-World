pipeline {
    agent {label 'jenkins-slave-mvn'}

    stages {
        stage('Build') {
            steps {
                echo 'Building'
                sh 'mvn compile'
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