pipeline {
  agent any

  environment {
    REGISTRY = "docker.io"
    DOCKERHUB_USER = "riadriri"
    IMAGE_TAG = "v1.0.${BUILD_NUMBER}"
  }

  parameters {
    string(name: 'ENVIRONMENT', defaultValue: 'dev', description: 'Environnement de déploiement (dev, staging, prod)')
  }

  stages {

    stage('Docker Build') {
      parallel {
        stage('Build cast-service') {
          steps {
            dir('charts/cast-service') {
              sh """
                echo "🚧 Building cast-service"
                docker build -t $DOCKERHUB_USER/cast-service:$IMAGE_TAG .
              """
            }
          }
        }

        stage('Build movie-service') {
          steps {
            dir('charts/movie-service') {
              sh """
                echo "🚧 Building movie-service"
                docker build -t $DOCKERHUB_USER/movie-service:$IMAGE_TAG .
              """
            }
          }
        }
      }
    }

    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh """
            echo "🔐 Logging into DockerHub"
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

            echo "🚀 Pushing cast-service image"
            docker push $DOCKERHUB_USER/cast-service:$IMAGE_TAG

            echo "🚀 Pushing movie-service image"
            docker push $DOCKERHUB_USER/movie-service:$IMAGE_TAG
          """
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          if (params.ENVIRONMENT == 'dev') {
            sh """
              echo "🔧 Deploying to DEV"
              helm upgrade --install cast-dev ./helm/cast-service --namespace dev --set image.tag=$IMAGE_TAG
              helm upgrade --install movie-dev ./helm/movie-service --namespace dev --set image.tag=$IMAGE_TAG
            """
          } else if (params.ENVIRONMENT == 'staging') {
            sh """
              echo "🚧 Deploying to STAGING"
              helm upgrade --install cast-staging ./helm/cast-service --namespace staging --set image.tag=$IMAGE_TAG
              helm upgrade --install movie-staging ./helm/movie-service --namespace staging --set image.tag=$IMAGE_TAG
            """
          } else if (params.ENVIRONMENT == 'prod') {
            input message: "⚠️ Confirmer le déploiement en PROD ?", ok: "Déployer"
            sh """
              echo "🚨 Deploying to PROD"
              helm upgrade --install cast-prod ./helm/cast-service --namespace prod --set image.tag=$IMAGE_TAG
              helm upgrade --install movie-prod ./helm/movie-service --namespace prod --set image.tag=$IMAGE_TAG
            """
          } else {
            error("❌ Environnement inconnu: ${params.ENVIRONMENT}")
          }
        }
      }
    }
  }

  post {
    always {
      echo "✅ Pipeline terminé avec le tag: $IMAGE_TAG"
    }
  }
}

