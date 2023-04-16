pipeline {
    agent any
    stages {
        stage('Fetch code') {
            steps {
                git branch: 'paac', url: 'https://github.com/adbordios/devopsprojects.git'
            }
        }
        stage('Build code') {
            steps {
                sh 'mvn install'
            }
        }
        stage('Test build') {
            steps {
                sh 'mvn test'
            }
        }
    }

    post {
        always {
            echo 'Build success'
        }
    }
}