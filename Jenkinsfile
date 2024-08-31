pipeline {
    agent any

    stages {
        // Checkout and work with the first repository
        stage('Checkout Catalog Repo') {
            steps {
                git url: 'git@github.com:DEL-ORG/s7michael-catalog.git', 
                    branch: 'main', 
                    credentialsId: 'github-ssh'
            }
        }

        stage('Unit Test') {
            agent {
                docker {
                    image 'golang:1.20'
                    args '-u root'
                }
            }
            steps {
                sh '''
                cd catalog
                go test 
                '''
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

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    cd catalog
                    docker build -t s7michael-catalog:${BUILD_NUMBER} .
                    '''
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            environment {
                DOCKER_CREDENTIALS_ID = 'del-docker-hub-auth'
                DOCKERHUB_REPO = 'devopseasylearning/s7michael-catalog'
            }
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        sh '''
                        docker tag s7michael-catalog:${BUILD_NUMBER} ${DOCKERHUB_REPO}:${BUILD_NUMBER}
                        docker push ${DOCKERHUB_REPO}:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }

        // Switch to the second repository
        stage('Checkout Helm Chart Repo') {
            steps {
                git url: 'git@github.com:DEL-ORG/catalog-s7michael.git', 
                    branch: 'main', 
                    credentialsId: 'github-ssh'
            }
        }

        stage('Update Helm Chart and Deploy') {
            steps {
                script {
                    // Update the image tag in the Helm chart's values.yaml file
                    sh '''
                    yq e '.image.tag = "${BUILD_NUMBER}"' -i /catalog
                    '''

                    // Deploy using Helm
                    sh '''
                    helm upgrade --install catalog /catalog2 --namespace s7michael
                    '''
                }
            }
        }
    }
}
