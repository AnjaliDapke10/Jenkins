pipeline {
    agent any

    stages {
        stage('Pull') {
            steps {
                git branch: 'main', url: 'https://github.com/SurajBele/studentdata.git'
                echo "Git Pull Successful"
            }
        }

        stage('Build') {
            steps {
                sh '/opt/apache-maven-3.9.11/bin/mvn clean package'
                echo "Build Successful"
            }
        }

        stage('Test') {
            steps {
                echo "Test Successful"
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploy Successful"
            }
        }
    }
}