pipeline {
    environment {
        DOCKER_ID = "webigeo"
        DOCKER_FRONT_IMAGE= "my-react"
        DOCKER_BACK_IMAGE = "my-django"
        DOCKER_TAG_TEST = "pre"
        DOCKER_TAG_TEST_PROD = "prod"
        KUBECONFIG = credentials("config")
    }
    agent {
        label 'Front_End'
    }

    stages {
        stage('Clean Up the kubernetes namespace pre & prod') {
            steps {
                script {
                    sh """
                    kubectl delete all --all -n pre
                    kubectl delete all --all -n prod
                    """
                }
            }
        }

        stage('Clean Up the kubernetes namespace prod') {
            agent {
                label 'Back_End'
            }
            environment {
                KUBECONFIG = credentials("config1")
            }
            steps {
                script {
                    sh """
                    kubectl delete all --all -n pre
                    kubectl delete all --all -n prod
                    """
                }
            }
        }

        stage('Clean Up Docker_Front_End') {
            steps {
                script {
                    sh """
                    #docker stop sqlite-container
                    #docker rm sqlite-container
                    #echo "docker rmi $DOCKER_ID/$DOCKER_FRONT_IMAGE:$DOCKER_TAG_TEST"
                    #echo "docker rmi $DOCKER_ID/$DOCKER_BACK_IMAGE:$DOCKER_TAG_TEST"
                    docker rmi $DOCKER_ID/$DOCKER_FRONT_IMAGE:$DOCKER_TAG_TEST || true
                    docker rmi $DOCKER_ID/$DOCKER_FRONT_IMAGE:$DOCKER_TAG_TEST_PROD || true
                    """
                }
            }
        }

        stage('Clean Up Docker_Back_End') {
             agent {
                label 'Back_End'
            }
            steps {
                script {
                    sh """
                    #docker stop sqlite-container
                    #docker rm sqlite-container
                    #echo "docker rmi $DOCKER_ID/$DOCKER_BACK_IMAGE:$DOCKER_TAG_TEST"
                    #echo "docker rmi $DOCKER_ID/$DOCKER_BACK_IMAGE:$DOCKER_TAG_TEST_PROD"
                    docker rmi $DOCKER_ID/$DOCKER_BACK_IMAGE:$DOCKER_TAG_TEST || true
                    docker rmi $DOCKER_ID/$DOCKER_BACK_IMAGE:$DOCKER_TAG_TEST_PROD || true
                    """
                }
            }
        }
        /*/
          
        stage('Pushing Front End image to DockerHub') {
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
                '''
                }
            }
        }
        
        stage('Pushing Back End image to DockerHub') {
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") 
            }

            steps {

                script {
                sh '''
                echo "docker login -u $DOCKER_ID -p $DOCKER_PASS"
                docker login -u $DOCKER_ID -p "yP?5Q>Ktp+YA%#_"
                docker push $DOCKER_ID/$DOCKER_BACK_IMAGE:$DOCKER_TAG_TEST
                '''
                }
            }
        }
        /*/

        stage('CD Deployment webigeo_Front in pre') {
            steps {
                script {
                    //git url: "https://github.com/WEBIGEO-ALTEN/WEBIGEO/", branch: 'master'
                    sh """
                    helm upgrade kubweb webigeo-front/ --values=webigeo-front/values-pre.yaml -n pre || helm install kubweb webigeo-front/ --values=webigeo-front/values-pre.yaml -n pre --create-namespace 
                    #kubectl apply -f pv.yml,pvc.yml,statefulset-sqlite.yml,service-sqlite.yml,deployment-react.yml,service-react.yml,app-prod-ingress.yml --namespace=test --kubeconfig=${KUBECONFIG}
                    """
                }
            }
        }
        
        stage('CD Deployment webigeo_Back in pre') {
            agent {
                label 'Back_End'
            }
            environment {
                KUBECONFIG = credentials("config1")
            }
            steps {
                script {
                    //git url: "https://github.com/WEBIGEO-ALTEN/WEBIGEO/", branch: 'master'
                    sh """
                    kubectl delete all --all -n pre
                    helm upgrade webigeo-pre webigeo-back/ --values=webigeo-back/values-pre.yaml -n pre || helm install kubweb webigeo-back/ --values=webigeo-back/values-pre.yaml -n pre
                    #kubectl apply -f pv.yml,pvc.yml,statefulset-sqlite.yml,service-sqlite.yml,deployment-react.yml,service-react.yml,app-prod-ingress.yml --namespace=test --kubeconfig=${KUBECONFIG}
                    """
                }
            }
        }

        stage('Testing Kubernetes services') {
            steps {
                script {
                    def url = "https://api.webigeo.dcpepper.cloudns.ph/"
            
                    def response = sh(script: "curl -s $url", returnStatus: true)
                    //echo "${responce}"
                    if (response == 0) {
                        echo "HTTP request to $url was successful"
                    } else {
                        error "HTTP request to $url failed with status code $response"
                    }
                }    
            }
        }

        stage('CD Deployment in Prod (Manually)') {
             steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }
                script { 
                    sh """
                    #helm upgrade kubweb webigeo/ --values=webigeo/values-prod.yaml -n prod || helm install kubweb webigeo/ --values=webigeo/values-prod.yaml -n prod --create-namespace
                    #kubectl apply -f statefulset-sqlite.yml,service-sqlite.yml,deployment-react.yml,service-react.yml,app-prod-ingress.yml --namespace=prod --kubeconfig=${KUBECONFIG}
                    """
                }
            }
        }

        stage('CD Deployment webigeo_Front in prod') {
            steps {
                script {
                    //git url: "https://github.com/WEBIGEO-ALTEN/WEBIGEO/", branch: 'master'
                    sh """
                    helm upgrade kubweb webigeo-front/ --values=webigeo-front/values-prod.yaml -n prod || helm install kubweb webigeo-front/ --values=webigeo-front/values-prod.yaml -n prod --create-namespace
                    """
                }
            }
        }
        
        stage('CD Deployment webigeo_Back in prod') {
            agent {
                label 'Back_End'
            }
            environment {
                KUBECONFIG = credentials("config1")
            }
            steps {
                script {
                    //git url: "https://github.com/WEBIGEO-ALTEN/WEBIGEO/", branch: 'master'
                    sh """
                    helm upgrade webigeo-prod webigeo-back/ --values=webigeo-back/values-prod.yaml -n prod || helm install kubweb webigeo-back/ --values=webigeo-back/values-prod.yaml -n prod
                    """
                }
            }
        }
    }
}

