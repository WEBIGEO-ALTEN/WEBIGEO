pipeline {
    environment {
        DOCKER_ID = "webigeo"
        DOCKER_IMAGE = "webigeo/my-sqlite-image"
        DOCKER_TAG = "latest"
        KUBECONFIG = credentials("config") 
    }
    agent any

    stages {
        stage('Docker Build') {
            steps {
                script {
                    sh """
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
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
                    #For the moments the ports are not open
                    """
                }
            }
        }

        stage('Deployment in webigeo') {
            steps {
                script {
                    sh """
                    #cp sqlite-deployment-dev.yaml sqlite-deployment.yaml
                    kubectl apply -f sqlite-deployment.yaml --namespace=webigeo --kubeconfig=$KUBECONFIG
                    """
                }
            }
        }
    }
}
