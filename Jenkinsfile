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
                    helm upgrade --install release ./helm --values=values-dev.yml --namespace dev
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
                    helm upgrade --install release ./helm --values=values-staging.yml --namespace staging
                    '''
                }
            }
        }


        stage('Deploy to QA') {
            environment {
                KUBECONFIG = credentials("config") // récupération du kubeconfig à partir du fichier secret
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp helm/values-qa.yaml values-qa.yml
                    cat values-qa.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values-qa.yml
                    helm upgrade --install release ./helm --values=values-qa.yml --namespace qa
                    '''
                }
            }
        }

        stage('Deploy to Prod') {
            when {
                branch 'master'
            }
            environment {
                KUBECONFIG = credentials("config") // récupération du kubeconfig à partir du fichier secret
            }
            steps {
                input 'Do you want to deploy to Prod?'
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp helm/values-prod.yaml values-prod.yml
                    cat values-prod.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values-prod.yml
                    helm upgrade --install release ./helm --values=values-prod.yml --namespace prod
                    '''
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
