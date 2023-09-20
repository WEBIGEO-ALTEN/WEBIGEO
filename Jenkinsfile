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
                    kubectl apply -f statefulset-sqlite.yml service-sqlite.yml deployment-react.yml service-react.yml app-ingress.yml --namespace=test --kubeconfig=$KUBECONFIG
                    """
                }
            }
        }
    }
}
