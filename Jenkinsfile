pipeline {
    environment {
        DOCKER_ID = "webigeo"
        DOCKER_IMAGE = "my-sqlite-image"
        DOCKER_TAG = "latest"
        KUBECONFIG = credentials("config") 
    }
    agent any

    stages {
        stage('Clean Up the kubernetes namespace test') {
            steps {
                script {
                    sh """
                    kubectl delete all --all -n test
                    """
                }
            }
        }
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

        stage('Deployment in webigeo') {
            steps {
                script {
                    sh """
                    kubectl apply -f statefulset-sqlite.yml,service-sqlite.yml,deployment-react.yml,service-react.yml,app-prod-ingress.yml --namespace=test --kubeconfig=$KUBECONFIG
                    """
                }
            }
        }

        stage('Testing the Kubernetes services') {
            steps {
                script {
                    def url = "https://api.webigeo.dcpepper.cloudns.ph/"
            
                    def response = sh(script: "curl -s $url", returnStatus: true)
            
                    if (response == 0) {
                        echo "HTTP request to $url was successful"
                    } else {
                        error "HTTP request to $url failed with status code $response"
            }
        }
    }
}

