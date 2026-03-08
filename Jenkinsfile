pipeline {
    agent any

    environment {
        DOCKER_USER="maxduessler"
        IMAGE_MOVIE="movie-service"
        IMAGE_CASE="cast-service"
        IMAGE_TAG="${BUILD_NUMBER}"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {

        stage("Checkout") {
            steps {
                checkout scm
            }
        }

        stage("Build") {
            steps {
                script {
                    sh """
                        docker build -t ${DOCKER_USER}/${IMAGE_MOVIE}:${IMAGE_TAG} movie-service/
                        docker build -t ${DOCKER_USER}/${IMAGE_CASE}:${IMAGE_TAG} cast-service/
                    """
                }
            }
        }

        stage("Push") {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh """
                        echo '${DOCKER_PASS}' | docker login -u '${DOCKER_USER}' --password-stdin
                        docker push ${DOCKER_USER}/${IMAGE_MOVIE}:${IMAGE_TAG}
                        docker push ${DOCKER_USER}/${IMAGE_CASE}:${IMAGE_TAG}
                        docker logout
                    """
                }
            }
        }

        stage("Deploy Dev") {
            steps {
                script {
                    sh """
                        helm upgrade --install app ./charts \
                        --namespace dev \
                        --set movie.image.tag=${IMAGE_TAG} \
                        --set cast.image.tag=${IMAGE_TAG}
                    """
                }
            }
        }

        stage("Deploy QA") {
            steps {
                script {
                    sh """
                        helm upgrade --install app ./charts \
                        --namespace qa \
                        --set movie.image.tag=${IMAGE_TAG} \
                        --set cast.image.tag=${IMAGE_TAG}
                    """
                }
            }
        }

        stage("Deploy Staging") {
            steps {
                script {
                    sh """
                        helm upgrade --install app ./charts \
                        --namespace staging \
                        --set movie.image.tag=${IMAGE_TAG} \
                        --set cast.image.tag=${IMAGE_TAG}
                    """
                }
            }
        }

        stage("Deploy Prod") {
            when {
                branch "master"
            }

            steps {
                script {
                    sh """
                        helm upgrade --install app ./charts \
                        --namespace prod \
                        --set movie.image.tag=${IMAGE_TAG} \
                        --set cast.image.tag=${IMAGE_TAG}
                    """
                }
            }
        }
    }
}
