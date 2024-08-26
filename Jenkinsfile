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

                    // Build and tag the Docker image
                    sh '''
                    cd catalog
                    docker build -t s7michael-catalog:${BUILD_NUMBER} .
                    '''
                    echo "Docker image built and tagged with build number: ${BUILD_NUMBER}"
                }
            }
        }

        stage('Push to DockerHub') {
            environment {
                DOCKER_CREDENTIALS_ID = 'del-docker-hub-auth' // Set this to your DockerHub credentials ID in Jenkins
                DOCKERHUB_REPO = 'devopseasylearning/s7michael-catalog' // Replace with your DockerHub repo
            }
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        sh '''
                        cd catalog
                        docker tag s7michael-catalog:${BUILD_NUMBER} ${DOCKERHUB_REPO}:${BUILD_NUMBER}
                        docker push ${DOCKERHUB_REPO}:${BUILD_NUMBER}
                        '''
                        echo "Docker image pushed to DockerHub with build number: ${BUILD_NUMBER}"
                    }
                }
            }
        }
    }
}
