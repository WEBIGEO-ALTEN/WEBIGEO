pipeline {
    environment {
        DOCKER_ID = "webigeo"
        DOCKER_FRONT_IMAGE= "my-react"
        DOCKER_BACK_IMAGE = "my-django"
        DOCKER_TAG_TEST = "latest"
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
                    echo "good"
                    #docker stop sqlite-container
                    #docker rm sqlite-container
                    #docker rmi $DOCKER_ID/$DOCKER_FRONT_IAMGE:$DOCKER_TAG_TEST
                    #docker rmi $DOCKER_ID/$DOCKER_BACK_IMAGE:$DOCKER_TAG_TEST
                    """
                }
            }
        }

        stage('Docker Build the Front End Image') {
            steps {
                script {
                    sh """
                    echo "docker build -t $DOCKER_ID/$DOCKER_FRONT_IMAGE:$DOCKER_TAG_TEST -f nginx-docker-image/Dockerfile ."
                    docker build -t $DOCKER_ID/$DOCKER_FRONT_IMAGE:$DOCKER_TAG_TEST -f nginx-docker-image/Dockerfile .
                    """
                }
            }
        }

        stage('Docker Build the Back End Image') {
            steps {
                script {
                    sh """
                    echo "docker build -t $DOCKER_ID/$DOCKER_BACK_IMAGE:$DOCKER_TAG_TEST -f sqlite-docker-image/Dockerfile ."
                    docker build -t $DOCKER_ID/$DOCKER_BACK_IMAGE:$DOCKER_TAG_TEST -f sqlite-docker-image/Dockerfile .
                    """
                }
            }
        }
        
         stage('Pushing the Front end image into DockerHub') {
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") 
            }

            steps {

                script {
                sh '''
                echo "docker login -u $DOCKER_ID -p $DOCKER_PASS"
                docker login -u $DOCKER_ID -p "yP?5Q>Ktp+YA%#_"
                docker push $DOCKER_ID/$DOCKER_FRONT_IMAGE:$DOCKER_TAG_TEST
                docker push $DCOKER_ID/$DOCKER_BACK_IMAGE:$DOCKER_TAG_TEST
                '''
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

