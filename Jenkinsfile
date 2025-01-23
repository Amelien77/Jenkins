pipeline {
    agent any

    environment {
        // Variables d'environnement pour Docker et Kubernetes
        DOCKER_IMAGE = 'amelien/cast_service'
        DOCKER_IMAGE_MOVIE = 'amelien/movie_service'
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

        stage('Run Tests') {
            steps {
                script {
                    // Ajoutez vos étapes de tests ici
                    echo 'Running tests...'
                    // Exemple : sh './run-tests.sh'
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
            steps {
                script {
                    // Déploiement dans l'environnement dev
                    echo 'Deploying to Dev...'
                    sh "helm upgrade --install release ./helm -f helm/values-dev.yaml -n $KUBE_NAMESPACE_DEV"
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    // Déploiement dans l'environnement staging
                    echo 'Deploying to Staging...'
                    sh "helm upgrade --install release ./helm -f helm/values-staging.yaml -n $KUBE_NAMESPACE_STAGING"
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                script {
                    // Déploiement dans l'environnement QA
                    echo 'Deploying to QA...'
                    sh "helm upgrade --install release ./helm -f helm/values-qa.yaml -n $KUBE_NAMESPACE_QA"
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
                    sh "helm upgrade --install release ./helm -f helm/values-prod.yaml -n $KUBE_NAMESPACE_PROD"
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
