pipeline {
    agent none
    environment {
        DOCKERHUB_USERNAME = "manrala"
        APP_NAME = "mann-gitops"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
        REGISTRY_CREDS = 'https://hub.docker.com'

    }
    stages {
        stage('Cleanup the workspace'){
            agent {
                docker {
                  image 'python:3.12.5-alpine3.19'
                }
            }
            steps {
                script {
                    echo "cleanup"
                    cleanWs()
                }
            }
        }
        stage('Checkout SCM'){
            agent {
                docker {
                  image 'python:3.12.5-alpine3.19'
                }
            }
            steps{
                echo "Downloading the script"
                git branch: 'main', credentialsId: 'GitHub', url: 'https://github.com/mannamnaveen/mann-gitops-code.git'
            }
        }
        stage('Build the docker image'){
            agent {
                docker {
                  image 'python:3.12.5-alpine3.19'
                }
            }
            steps{
                script{
                    echo "Build the docker image."
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }
        stage('Push the image'){
            agent {
                docker {
                  image 'python:3.12.5-alpine3.19'
                }
            }
            steps{
                script{
                    docker.withRegistry('docker'){
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }
        stage('Delete the docker image'){
            steps{
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }
        stage('Updating the K8S deployment.yaml file'){
            steps{
                sh "cat deployment.yml"
                sh "sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yml "
                sh "cat deployment.yml"
            }
        }
        stage('Push the changes to main branch'){
            steps{
                script{
                    sh """
                      git config --global user.name "mannamnaveen"
                      git config --global user.email "mina@naveenmannam.com"
                      git add deployment.yml
                      git commit -m 'Updated the image tag in deployment.yml'"""
                      withCredentials([usernamePassword(credentialsId: 'GitHub', passwordVariable: 'password', usernameVariable: 'username')]) {
                      sh "git push https://$username:$password@github.com/mannamnaveen/mann-gitops-code.git main"
                    }
                }
            }
        }
    }
}
