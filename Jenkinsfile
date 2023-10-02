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

        stage('Clean Up Docker Front End') {
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

        stage('Clean Up Docker Back End') {
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

        stage('CD Deployment webigeo Front in pre') {
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
        
        stage('CD Deployment webigeo Back in pre') {
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
                    helm upgrade webigeo-pre webigeo-back/ --values=webigeo-back/values-pre.yaml -n pre || helm install webigeo-pre webigeo-back/ --values=webigeo-back/values-pre.yaml -n pre
                    #kubectl apply -f pv.yml,pvc.yml,statefulset-sqlite.yml,service-sqlite.yml,deployment-react.yml,service-react.yml,app-prod-ingress.yml --namespace=test --kubeconfig=${KUBECONFIG}
                    """
                }
            }
        }

        stage('Testing Kubernetes services') {
            steps {
                script {
                    def url = "https://webigeo-pre.dcpepper.cloudns.ph/home"
            
                    def response = sh(script: "timeout 30 curl -i $url", returnStatus: true)
                    echo "${response}"
                    if (response != 0) {
                        error "HTTP request to $url failed, check the URL and try again."
                    } else {
                        def statusCode = sh(script: "curl -s -o /dev/null -w '%{http_code}' $url", returnStatus: true)
                        echo"This is the test case : ${statusCode}"    
                        if (statusCode == 0) {
                            echo "HTTP request to $url was successful. Status code: $statusCode"
                        } else {
                            error "HTTP request to $url failed with status code $statusCode"
                        }
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
                    if (env.BRANCH_NAME == 'master') {
                        echo "this enters into if statement"
                        def dir = sh(script: "pwd", returnStdout: true)
                        echo "the directory is: ${dir}"
                    } else {
                        error "Something is fishy"
                        }
                }
            }
        }

        stage('CD Deployment webigeo Front in prod') {
            steps {
                script {
                    //git url: "https://github.com/WEBIGEO-ALTEN/WEBIGEO/", branch: 'master'
                    sh """
                    helm upgrade kubweb1 webigeo-front/ --values=webigeo-front/values-prod.yaml -n prod || helm install kubweb1 webigeo-front/ --values=webigeo-front/values-prod.yaml -n prod --create-namespace
                    """
                }
            }
        }
        
        stage('CD Deployment webigeo Back in prod') {
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
                    helm upgrade webigeo-prod webigeo-back/ --values=webigeo-back/values-prod.yaml -n prod || helm install webigeo-prod webigeo-back/ --values=webigeo-back/values-prod.yaml -n prod
                    """
                }
            }
        }
    }
}

