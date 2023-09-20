pipeline {
    environment {
        DOCKER_ID = "webigeo"
        DOCKER_IMAGE = "my-sqlite-image"
        DOCKER_TAG_TEST = "test"
        KUBECONFIG = credentials("config") 
        DOCKERCONFIG = credentials("DOCKER_HUB_PASS")
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
                    #docker stop sqlite-container
                    #docker rm sqlite-container
                    #docker rmi $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    """
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    sh """
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG_TEST -f sqlite-docker-image/Dockerfile .
                    """
                }
            }
        }

        stage('Pushing the image into DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://hub.docker.com/',DOCKERCONFIG){
                        dockerImage.push("test")
                    }
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
    }
}

