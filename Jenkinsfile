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
                git url: 'git@github.com:DEL-ORG/s7michael-deployment.git', 
                    branch: 'dev', 
                    credentialsId: 'github-ssh'
            }
        }

        stage('Update Helm Chart') {
            steps {
                script {
                    // Update the image tag in the Helm chart's values.yaml file
                    sh '''
                    yq e '.catalog.tag = "${BUILD_NUMBER}"' -i ./chart/dev-values.yaml
                    '''
                }
            }
        }

        stage('Commit and Push Changes') {
            steps {
                script {
                    // Commit and push the changes to the second repository
                    sh '''
                    git config user.email "michaelsobamowo@gmail.com"
                    git add -A
                    git commit -m "Update image tag to ${BUILD_NUMBER}"
                    git push --set-upstream origin dev
                    '''
                }
            }
        }
    }
}






