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

        stage("Cleanup old services") {
            steps {
                sh """
                kubectl delete svc movies-fastapiapp -n dev || true
                kubectl delete svc movies-fastapiapp -n qa || true
                kubectl delete svc movies-fastapiapp -n staging || true
                kubectl delete svc movies-fastapiapp -n prod || true
                """
            }
        }

        stage("Create namespaces") {
                steps {
                    sh """
                        kubectl create namespace dev || true
                        kubectl create namespace qa || true
                        kubectl create namespace staging || true
                        kubectl create namespace prod || true
                    """
                }
            }

        stage("Deploy Dev") {
            steps {
                script {
                    sh """
                        helm upgrade --install app ./charts \
                        --namespace dev \
                        --set movie.image.tag=${IMAGE_TAG} \
                        --set cast.image.tag=${IMAGE_TAG} \
                        --set service.nodePort=30007
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
                        --set cast.image.tag=${IMAGE_TAG}\
                        --set service.nodePort=30008
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
                        --set cast.image.tag=${IMAGE_TAG}\
                        --set service.nodePort=30009
                    """
                }
            }
        }

        stage("Deploy Prod") {

            steps {
                script {
                    sh """
                        helm upgrade --install app ./charts \
                        --namespace prod \
                        --set movie.image.tag=${IMAGE_TAG} \
                        --set cast.image.tag=${IMAGE_TAG}\
                        --set service.nodePort=30010
                    """
                }
            }
        }
    }
}
