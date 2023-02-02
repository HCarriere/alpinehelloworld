/* import shared library */
@Library('chocoapp-slack-share-library')_

pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        APP_EXPOSED_PORT = "80"
        IMAGE_TAG = "latest"
        STAGING = "ulrich-staging"
        PRODUCTION = "ulrich-prod"
        DOCKERHUB_ID = "choco1992"
        DOCKERHUB_PASSWORD = credentials('dockerhub_password')
        APP_NAME = "ulrich"
        STG_API_ENDPOINT = "100.26.154.3:1993"
        STG_APP_ENDPOINT = "100.26.154.3:8080"
        PROD_API_ENDPOINT = "100.26.154.3:1993"
        PROD_APP_ENDPOINT = "100.26.154.3:8080"
        INTERNAL_PORT = "5000"
        EXTERNAL_PORT = "${PORT_EXPOSED}"
        CONTAINER_IMAGE = "${DOCKERHUB_ID}/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    agent none
    stages {
       stage('Build image') {
           agent any
           steps {
              script {
                sh 'docker build -t ${votre_id_dockerhub}/${IMAGE_NAME}:${IMAGE_TAG} .'
              }
           }
       }
       stage('Run container based on builded image') {
          agent any
          steps {
            script {
              sh '''
docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} ${votre_id_dockerhub}/${IMAGE_NAME}:${IMAGE_TAG} 
              '''
             }
          }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''
curl http://172.17.0.1 | grep -q "Hello world!"
                '''
              }
           }
       }
       stage('Clean container') {
          agent any
          steps {
             script {
               sh '''
docker stop ${IMAGE_NAME}
docker rm  ${IMAGE_NAME}
               '''
             }
          }
      }

      stage ('Login and Push Image on docker hub') {
          agent any
          steps {
             script {
               sh '''
echo $DOCKERHUB_PASSWORD_PSW | docker login -u $DOCKERHUB_PASSWORD_USR --password-stdin
docker push ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
               '''
             }
          }
      }

      stage('STAGING - Deploy app') {
      agent any
      steps {
          script {
            sh """
              echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}00\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
              curl -v -X POST http://${STG_API_ENDPOINT}/staging -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
            """
          }
        }
     
     }
     stage('PROD - Deploy app') {
       when {
           expression { GIT_BRANCH == 'origin/main' }
       }
     agent any

       steps {
          script {
            sh """
              echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
              curl -v -X POST http://${PROD_API_ENDPOINT}/prod -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
            """
          }
       }
     }
  }
}