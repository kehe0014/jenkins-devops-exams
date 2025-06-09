pipeline {

    environment {
        DOCKER_ID = "tdksoft"
        IMAGE_TAG = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        HELM_CHART_PATH = './helm'
        HELM_RELEASE_NAME = "my-python-app-${env.BRANCH_NAME}"
        K8S_NAMESPACE = "default"
    }

    agent any

    stages {

    stage('Build Docker Image') { // Build the Docker image using docker-compose
            steps {
                script {
                    sh """
                    docker compose build
                    sleep 6
                    """
                }
            }
    }

    stage('Push Docker Image') { // Push the Docker image to Docker Hub
            when {
                expression { env.BRANCH_NAME in ['main', 'dev'] }
            }
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh """
                        docker login -u ${DOCKER_ID} -p ${DOCKER_PASS}
                        docker compose push 
                    """ 
                }
            }
        }
/*
        stage('Choose Environment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.TARGET_ENV = 'prod'
                        env.K8S_NAMESPACE = 'production'
                    } else if (env.BRANCH_NAME == 'dev') {                    
                        env.TARGET_ENVS = ['dev', 'staging', 'qa']
                        env.K8S_NAMESPACE = 'development' /
                    } else {
                        env.TARGET_ENVS = ['dev']
                        env.K8S_NAMESPACE = 'development'
                    }
                }
            }
        }

*/  }      
    post {
        always {
            echo "Pipeline finished for branch ${env.BRANCH_NAME}"
        }
        success {
            echo "Pipeline successful!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
