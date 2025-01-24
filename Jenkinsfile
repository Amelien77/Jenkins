pipeline {
    agent any

    environment {
        // Variables d'environnement pour Docker et Kubernetes
        DOCKER_IMAGE_CAST = 'ameliendevops/cast-service'
        DOCKER_IMAGE_MOVIE = 'ameliendevops/movie-service'
        DOCKER_TAG = "v.${BUILD_ID}.0"
        KUBE_NAMESPACE_DEV = 'dev'
        KUBE_NAMESPACE_STAGING = 'staging'
        KUBE_NAMESPACE_QA = 'qa'
        KUBE_NAMESPACE_PROD = 'prod'
    }

    stages {
        // Étape de construction des images Docker
        stage('Build Docker Images') {
            steps {
                script {
                    echo 'Building Docker images...'
                    // Build des images Docker pour les services
                    sh "docker build -t $DOCKER_IMAGE_CAST:$DOCKER_TAG ./cast-service"
                    sh "docker build -t $DOCKER_IMAGE_MOVIE:$DOCKER_TAG ./movie-service"
                }
            }
        }

        // Étape pour exécuter les images Docker localement (facultatif mais utile pour vérifier que tout fonctionne)
        stage('Run Docker Containers') {
            steps {
                script {
                    echo 'Running Docker containers for testing...'
                    // Run des images Docker pour tester localement
                    sh "docker run -d --name cast-service $DOCKER_IMAGE_CAST:$DOCKER_TAG"
                    sh "docker run -d --name movie-service $DOCKER_IMAGE_MOVIE:$DOCKER_TAG"
                }
            }
        }

        // Étape pour pousser les images Docker sur DockerHub
        stage('Push Docker Images') {
            steps {
                script {
                    echo 'Pushing Docker images to DockerHub...'
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Connexion à DockerHub
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                        // Pousser les images vers DockerHub
                        sh "docker push $DOCKER_IMAGE_CAST:$DOCKER_TAG"
                        sh "docker push $DOCKER_IMAGE_MOVIE:$DOCKER_TAG"
                    }
                }
            }
        }

        // Étape de déploiement sur l'environnement Dev
        stage('Deploy to Dev') {
            environment {
                KUBECONFIG = credentials("config") // Assurez-vous que la configuration Kube est stockée dans Jenkins
            }
            steps {
                script {
                    echo 'Deploying to Dev environment...'
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp helm/values-dev.yaml values-dev.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values-dev.yml
                    helm upgrade --install release ./helm --values=values-dev.yml --namespace $KUBE_NAMESPACE_DEV
                    '''
                }
            }
        }

        // Étape de déploiement sur l'environnement Staging
        stage('Deploy to Staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    echo 'Deploying to Staging environment...'
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp helm/values-staging.yaml values-staging.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values-staging.yml
                    helm upgrade --install release ./helm --values=values-staging.yml --namespace $KUBE_NAMESPACE_STAGING
                    '''
                }
            }
        }

        // Étape de déploiement sur l'environnement QA
        stage('Deploy to QA') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    echo 'Deploying to QA environment...'
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp helm/values-qa.yaml values-qa.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values-qa.yml
                    helm upgrade --install release ./helm --values=values-qa.yml --namespace $KUBE_NAMESPACE_QA
                    '''
                }
            }
        }

        // Déploiement en Production uniquement pour la branche master
        stage('Deploy to Prod') {
            when {
                branch 'master'
            }
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                input 'Do you want to deploy to Prod?'
                script {
                    echo 'Deploying to Prod environment...'
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp helm/values-prod.yaml values-prod.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values-prod.yml
                    helm upgrade --install release ./helm --values=values-prod.yml --namespace $KUBE_NAMESPACE_PROD
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed, please check the logs!'
        }
    }
}
