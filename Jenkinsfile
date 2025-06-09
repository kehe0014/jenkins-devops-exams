pipeline {
    agent any

    environment {
        DOCKER_ID = "tdksoft"
        IMAGE_NAME = "${DOCKER_ID}/jenkins-devops-eval"
        IMAGE_TAG = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        HELM_CHART_PATH = './helm'
        HELM_RELEASE_NAME = "my-python-app-${env.BRANCH_NAME}"
        K8S_NAMESPACE = "default"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
                script {
                    env.BRANCH_NAME = env.GIT_BRANCH.replace('origin/', '')
                    echo "Building branch: ${env.BRANCH_NAME}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    echo "Pushing Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh """
                        docker login -u ${DOCKER_ID} -p ${DOCKER_PASS}
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

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