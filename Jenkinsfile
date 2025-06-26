pipeline {
    environment {
        DOCKER_ID = 'scornedevops'
        DOCKER_IMAGE = 'jenkins_devops_exam'
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any
    stages {
      // === CAST SERVICE ===

        stage('Build cast-service') {
            steps {
                dir('cast-service') {
                    sh "docker build -t $DOCKER_ID/cast-service:$DOCKER_TAG ."
                }
            }
        }

        stage('Run & Test cast-service') {
            steps {
                script {
                    sh """
                      docker rm -f cast-service || true
                      docker run -d -p 8080:80 --name cast-service $DOCKER_ID/cast-service:$DOCKER_TAG
                      sleep 10
                      curl -f localhost:8080 || (echo 'Cast-service failed test' && exit 1)
                      docker rm -f cast-service
                  """
                }
            }
        }

        stage('Push cast-service') {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_HUB_PASS', variable: 'DOCKER_PASS')]) {
                    sh """
                      docker login -u $DOCKER_ID -p $DOCKER_PASS
                      docker push $DOCKER_ID/cast-service:$DOCKER_TAG
                  """
                }
            }
        }

      // === MOVIE SERVICE ===

        stage('Build movie-service') {
            steps {
                dir('movie-service') {
                    sh "docker build -t $DOCKER_ID/movie-service:$DOCKER_TAG ."
                }
            }
        }

        stage('Run & Test movie-service') {
            steps {
                script {
                    sh """
                      docker rm -f movie-service || true
                      docker run -d -p 8081:80 --name movie-service $DOCKER_ID/movie-service:$DOCKER_TAG
                      sleep 10
                      curl -f localhost:8081 || (echo 'Movie-service failed test' && exit 1)
                      docker rm -f movie-service
                  """
                }
            }
        }

        stage('Push movie-service') {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_HUB_PASS', variable: 'DOCKER_PASS')]) {
                    sh """
                      docker login -u $DOCKER_ID -p $DOCKER_PASS
                      docker push $DOCKER_ID/movie-service:$DOCKER_TAG
                  """
                }
            }
        }

        stage('Deploiement en dev') {
            environment
        {
                KUBECONFIG = credentials('config') // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                    sh '''
                rm -Rf .kube
                mkdir .kube
                cat $KUBECONFIG > .kube/config
                cp charts/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app jenkins_devops_exam --values=values.yml --namespace dev
                '''
                }
            }
        }
        stage('Deploiement en qa') {
            environment
        {
                KUBECONFIG = credentials('config') // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                    sh '''
              rm -Rf .kube
              mkdir .kube
              cat $KUBECONFIG > .kube/config
              cp charts/values.yaml values.yml
              cat values.yml
              sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
              helm upgrade --install app jenkins_devops_exam --values=values.yml --namespace qa
              '''
                }
            }
        }
        stage('Deploiement en staging') {
            environment
        {
                KUBECONFIG = credentials('config') // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                    sh '''
              rm -Rf .kube
              mkdir .kube
              cat $KUBECONFIG > .kube/config
              cp charts/values.yaml values.yml
              cat values.yml
              sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
              helm upgrade --install app jenkins_devops_exam --values=values.yml --namespace staging
              '''
                }
            }
        }
        stage('Deploiement en prod') {
            environment
        {
                KUBECONFIG = credentials('config') // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                    // Create an Approval Button with a timeout of 15minutes.
                    // this require a manuel validation in order to deploy on production environment
                    timeout(time: 15, unit: 'MINUTES') {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                    script {
                        sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app jenkins_devops_exam --values=values.yml --namespace prod
                    '''
                    }
            }
        }
    }
}

