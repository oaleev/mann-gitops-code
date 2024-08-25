pipeline {
    agent any
    environment {
        DOCKERHUB_USERNAME = "manrala"
        APP_NAME = "mann-gitops"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
        REGISTRY_CREDS = 'https://hub.docker.com'

    }
    stages {
        stage('Cleanup the workspace'){
            steps {
                script {
                    echo "cleanup"
                    cleanWs()
                }
            }
        }
        stage('Checkout SCM'){
            steps{
                echo "Downloading the script"
                git branch: 'main', credentialsId: 'GitHub', url: 'https://github.com/mannamnaveen/mann-gitops-code.git'
            }
        }
        stage('Build the docker image'){
            steps{
                script{
                    echo "Build the docker image"
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }
        stage('Push the image'){
            steps{
                script{
                    docker.withRegistry('docker'){
                        sh "push"
                    }
                }
            }
        }
        stage('Delete the docker image'){
            steps{
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker rmi ${IMAGE_NAME}:latest"
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
