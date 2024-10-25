pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/your-username/your-repository.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('nodejs-app')
                }
            }
        }
        stage('Run Docker Container') {
            steps {
                script {
                    docker.withRun('nodejs-app') {
                        sh 'echo "App is running"'
                    }
                }
            }
        }
    }
}
