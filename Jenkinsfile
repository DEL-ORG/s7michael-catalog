pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'git@github.com:DEL-ORG/s7michael-catalog.git', 
                    branch: 'main', 
                    credentialsId: 'github-ssh'
            }
        }

        stage('Unit Test') {
            agent {
                docker {
                    image 'golang:1.20' // Docker image with Go installed
                    args '-u root' // Run as root
                }
            }
            steps {
                sh 'go test' // Run Go tests with verbose output
            }
        }
    }
}
