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
                sh '''
                cd catalog
                go test -v ./...
                ''' // Run Go tests with verbose output
            }
        }

        stage('SonarQube Analysis') {
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli:4.7.0'
                }
            }
            environment {
                CI = 'true'
                scannerHome = '/opt/sonar-scanner'
            }
            steps {
                withSonarQubeEnv('Sonar') {
                    sh '''
                    cd catalog
                    ${scannerHome}/bin/sonar-scanner
                    '''
                }
            }
        }
    }
}
