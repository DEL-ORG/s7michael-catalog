pipeline {
    agent any

    options {
        // Keep only the last 10 builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

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

        // Switch to the second repository and update Helm chart
        stage('Update and Push Helm Chart') {
            steps {
                script {
                    // Checkout the second repository
                    git url: 'git@github.com:DEL-ORG/s7michael-deployment.git', 
                        branch: 'dev', 
                        credentialsId: 'github-ssh'
                    
                    // Update the image tag in the Helm chart's values.yaml file and push changes
                    sh '''
                    yq e '.catalog.tag = "${BUILD_NUMBER}"' -i ./chart/dev-values.yaml
                    git config user.email "michaelsobamowo@gmail.com"
                    git config user.name "michael-ayo"
                    git add -A
                    git commit -m "Update image tag to ${BUILD_NUMBER}" || echo "No changes to commit"
                    git push --set-upstream origin dev || echo "Push failed"
                    '''
                }
            }
        }
    }
}
