pipeline {
    environment {
        DOCKER_ID = "webigeo"
        DOCKER_IMAGE = "my-sqlite-image"
        DOCKER_TAG = "latest"
        KUBECONFIG = credentials("config") 
    }
    agent any

    stages {
        stage('Clean Up Docker') {
            steps {
                script {
                    sh """
                    docker stop sqlite-container
                    docker rm sqlite-container
                    docker rmi $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    """
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    sh """
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG -f sqlite-docker-image/Dockerfile .
                    """
                }
            }
        }

        stage('Docker Run SQLite') {
            steps {
                script {
                    sh """
                    docker run -d -p 8000:8000 --name sqlite-container $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    """
                }
            }
        }

        stage('Test SQLite Container') {
            steps {
                script {
                    sh """
                    #For the moments the project is running on other kubernetes  
                    """
                }
            }
        }

        stage('Deployment in webigeo') {
            steps {
                script {
                    sh """
                    #cp sqlite-deployment-dev.yaml sqlite-deployment.yaml
                    kubectl apply -f sqlite-statefulset.yaml --namespace=webigeo --kubeconfig=$KUBECONFIG
                    """
                }
            }
        }
    }
}
