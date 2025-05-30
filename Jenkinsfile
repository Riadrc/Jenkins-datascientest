pipeline {
  agent any

  environment {
    REGISTRY = "docker.io"
    DOCKERHUB_USER = "riadriri"
    IMAGE_TAG = "v1.0.${BUILD_NUMBER}"
  }

  stages {

    stage('Docker Build') {
      parallel {
        stage('Build cast-service') {
          steps {
            dir('cast-service') {
              sh "docker build -t $DOCKERHUB_USER/cast-service:$IMAGE_TAG ."
            }
          }
        }

        stage('Build movie-service') {
          steps {
            dir('movie-service') {
              sh "docker build -t $DOCKERHUB_USER/movie-service:$IMAGE_TAG ."
            }
          }
        }
      }
    }

    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push $DOCKERHUB_USER/cast-service:$IMAGE_TAG
            docker push $DOCKERHUB_USER/movie-service:$IMAGE_TAG
          '''
        }
      }
    }

    stage('Deploy to Dev') {
      steps {
        withCredentials([file(credentialsId: 'kubeconfig-credentials-id', variable: 'KUBECONFIG_FILE')]) {
          sh '''
            export KUBECONFIG=$KUBECONFIG_FILE
            helm upgrade --install cast-dev ./helm/cast-service --namespace dev --set image.tag=$IMAGE_TAG
            helm upgrade --install movie-dev ./helm/movie-service --namespace dev --set image.tag=$IMAGE_TAG
          '''
        }
      }
    }

    stage('Deploy to Staging') {
      steps {
        withCredentials([file(credentialsId: 'kubeconfig-credentials-id', variable: 'KUBECONFIG_FILE')]) {
          sh '''
            export KUBECONFIG=$KUBECONFIG_FILE
            helm upgrade --install cast-staging ./helm/cast-service --namespace staging --set image.tag=$IMAGE_TAG
            helm upgrade --install movie-staging ./helm/movie-service --namespace staging --set image.tag=$IMAGE_TAG
          '''
        }
      }
    }

    stage('Deploy to Prod') {
      when {
        branch 'master'
      }
      input {
        message "Déployer en production ?"
      }
      steps {
        withCredentials([file(credentialsId: 'kubeconfig-credentials-id', variable: 'KUBECONFIG_FILE')]) {
          sh '''
            export KUBECONFIG=$KUBECONFIG_FILE
            helm upgrade --install cast-prod ./helm/cast-service --namespace prod --set image.tag=$IMAGE_TAG
            helm upgrade --install movie-prod ./helm/movie-service --namespace prod --set image.tag=$IMAGE_TAG
          '''
        }
      }
    }
  }

  post {
    always {
      echo "Pipeline terminé avec le tag: $IMAGE_TAG"
    }
  }
}

