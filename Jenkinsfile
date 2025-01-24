pipeline {
    agent any

    environment {
        // Variables d'environnement pour Docker et Kubernetes
        DOCKER_IMAGE = 'ameliendevops/movie-cast-service'
        DOCKER_IMAGE_MOVIE = 'ameliendevops/movie-cast-service'
        DOCKER_TAG = "v.${BUILD_ID}.0"
        KUBE_NAMESPACE_DEV = 'dev'
        KUBE_NAMESPACE_STAGING = 'staging'
        KUBE_NAMESPACE_QA = 'qa'
        KUBE_NAMESPACE_PROD = 'prod'
    }

    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    // Build des images Docker pour les services
                    echo 'Building Docker images...'
                    sh 'docker build -t $DOCKER_IMAGE:latest ./cast-service'
                    sh 'docker build -t $DOCKER_IMAGE_MOVIE:latest ./movie-service'
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    // Push des images Docker vers DockerHub
                    echo 'Pushing Docker images to DockerHub...'
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                        sh "docker push $DOCKER_IMAGE:latest"
                        sh "docker push $DOCKER_IMAGE_MOVIE:latest"
                    }
                }
            }
        }

        stage('Deploy to Dev') {
            environment {
                KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp helm/values-dev.yaml values-dev.yml
                    cat values-dev.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values-dev.yml
                    helm upgrade --install release ./kubernetes --values=values-dev.yml --namespace dev
                    '''
                }
            }
        }

        stage('Deploy to Staging') {
            environment {
                KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp helm/values-staging.yaml values-staging.yml
                    cat values-staging.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values-staging.yml
                    helm upgrade --install release ./kubernetes --values=values-staging.yml --namespace staging
                    '''
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                script {
                    // Déploiement dans l'environnement QA
                    echo 'Deploying to QA...'
                    withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                        // Copier le fichier kubeconfig dans un répertoire temporaire
                        sh "cp $KUBECONFIG /tmp/kubeconfig.yaml"
                        sh "kubectl --kubeconfig=/tmp/kubeconfig.yaml apply -f ./k8s/qa-deployment.yaml"
                        sh "helm upgrade --install release ./helm -f helm/values-qa.yaml -n $KUBE_NAMESPACE_QA"
                    }
                }
            }
        }

        stage('Deploy to Prod') {
            when {
                branch 'master'
            }
            steps {
                input 'Do you want to deploy to Prod?'
                script {
                    // Déploiement en prod
                    echo 'Deploying to Prod...'
                    withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                        // Copier le fichier kubeconfig dans un répertoire temporaire
                        sh "cp $KUBECONFIG /tmp/kubeconfig.yaml"
                        sh "kubectl --kubeconfig=/tmp/kubeconfig.yaml apply -f ./k8s/prod-deployment.yaml"
                        sh "helm upgrade --install release ./helm -f helm/values-prod.yaml -n $KUBE_NAMESPACE_PROD"
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed, please check the logs!'
        }
    }
}
