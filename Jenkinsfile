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
                    echo "Deploying in the Prod"
                }
            }
        }

        stage('Checkout Dev Branch for front') {
            steps {
                // Checkout the dev branch
                script {
                    checkout([$class: 'GitSCM', 
                        branches: [[name: 'dev']], 
                        doGenerateSubmoduleConfigurations: false, 
                        extensions: [], 
                        submoduleCfg: [], 
                        userRemoteConfigs: [[url: 'https://github.com/WEBIGEO-ALTEN/WEBIGEO_FRONT.git']]
                    ])
                }
            }
        }

        stage('Merge Dev into Main for Front') {
            steps {
                // Navigate to the local main branch
                sh 'git checkout main'
                
                // Merge the dev branch into main
                sh 'git merge origin/dev'
                
                sh 'git config --global user.name goli-sateesh-6011'
                sh 'git config --global user.email goli.sateesh@gmail.com'
                
                //sh 'git config credential.helper "store --file=$HOME/.git-credentials"'

                sh 'git push https://goli-sateesh-6011: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@github.com/WEBIGEO-ALTEN/WEBIGEO_FRONT.git main'

            }
        }

        stage('Checkout Dev Branch for back') {
            agent {
                label 'Back_End'
            }
            environment {
                KUBECONFIG = credentials("config1")
            }
            steps {
                // Checkout the dev branch
                script {
                    checkout([$class: 'GitSCM', 
                        branches: [[name: 'dev']], 
                        doGenerateSubmoduleConfigurations: false, 
                        extensions: [], 
                        submoduleCfg: [], 
                        userRemoteConfigs: [[url: 'https://github.com/WEBIGEO-ALTEN/WEBIGEO_BACK.git']]
                    ])
                }
            }
        }

        stage('Merge Dev into Main for back') {
            agent {
                label 'Back_End'
            }
            environment {
                KUBECONFIG = credentials("config1")
            }
            steps {
                // Navigate to the local main branch
                sh 'git checkout main'
                
                // Merge the dev branch into main
                sh 'git merge origin/dev'withCredentials([string(credentialsId: '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', variable: 'GITHUB_PAT')]) {
                sh """
                    git push https://goli-sateesh-6011:$GITHUB_PAT@github.com/WEBIGEO-ALTEN/WEBIGEO_BACK.git main
                """
                
                sh 'git config --global user.name goli-sateesh-6011'
                sh 'git config --global user.email goli.sateesh@gmail.com'
                
                sh 'git config credential.helper "store --file=$HOME/.git-credentials"'

                // Push the changes to the remote main branch
                sh 'git push https://goli-sateesh-6011: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@github.com/WEBIGEO-ALTEN/WEBIGEO_BACK.git main'
                }
            }
        }

        stage('CD Deployment in Prod') {
             steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                script {
                    echo "Branch name is: ${env.BRANCH_NAME}"
                    echo "Triggering another pipeline job"
                    build job: 'WEBIGEO_BACK_PROD', parameters: [string(name: 'param1', value: "value1")], wait: true
                    build job: 'WEBIGEO_FRONT_PROD', parameters: [string(name: 'param1', value: "value1")], wait: true
                }
            }
        }

    }
}