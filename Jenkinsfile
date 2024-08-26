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
                    image 'sonarsource/sonar-scanner-cli:11.0'
                    args '-u root'
                }
            }
            environment {
                CI = 'true'
                scannerHome = '/opt/sonar-scanner'
            }
            steps {
                withSonarQubeEnv('Sonar') {
                    sh '''
                    ${scannerHome}/bin/sonar-scanner
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    // Ensure Docker is available
                    sh 'docker --version'

                    // Print debug information
                    sh '''
                    echo "Current directory:"
                    pwd
                    echo "Listing files:"
                    ls -al
                    cd catalog
                    echo "Listing files in catalog:"
                    ls -al
                    '''

                    // Build and tag the Docker image
                    sh '''
                    cd catalog
                    docker build -t s7michael-catalog:${BUILD_NUMBER} .
                    '''
                }
            }
        }
    }
}
