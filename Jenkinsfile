/* import shared library */
@library('chocoapp-slack-share-library')_

pipeline{
  environment {
    IMAGE_NAME = "staticwebsite"
    APP_ CONTAINER_PORT = "5000"
    APP_EXPOSED_PORT = "80"
    IMAGE_TAG = "latest"
    STAGING = "ezz-statging"
    PRODUCTION = " ezz-prod"
    DOCKERHUB_ID = "trainingta"
    DOCKERHUB_PASSWORD = crendentials('dockerhub_password')
  
  }
    
agent none
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
                  sh 'docker build -t ${OCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG .'
                }
             }
          }
       

         stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    docker ps -a | grep -i $IMAGE_NAME && docker rm -f $IMAGE_NAME
                    docker run --name $IMAGE_NAME -d -p $APP_EXPOSED_PORT:$APP_ CONTAINER_PORT -e PORT=$APP_ CONTAINER_PORT ${OCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
                    sleep 5
                 '''
               }
            }
          }
       
          stage('Test image') {
           agent any
           steps {
              script {
                sh '''
                    curl http://192.168.56.7 | grep -i "Dimension"
                '''
              }
           }
          }
       
          stage('Clean Container') {
          agent any
          steps {
             script {
               sh '''
                 docker stop $IMAGE_NAME
                 docker rm $IMAGE_NAME
               '''
                    }
                }
          }
          
          stage('Login and Push Image on docker hub') {
          agent any
          steps {
             script {
               sh '''
                 docker $ DOCKERHUB_PASSWORD | docker login -u $ DOCKERHUB_ID --password-stdin
                 docker push  ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
               '''
                    }
                }
          }
         
          stage('Push image in staging and deploy it') {
             when {
              expression { GIT_BRANCH == 'origin/master' }
            }
            agent {
              docker {image 'franela/dind'}
            }
             environment {
                 HEROKU_API_KEY = credentials('heroku_api_key')
             }  
           steps {
              script {
                sh '''
                    apk --no caher and npm
                    npm install -g heroku 
                    heroku container:login
                    heroku create $STAGING || echo "project already exist"
                    heroku container:push -a $STAGING web
                    heroku container:release -a $STAGING web
            '''
          }
        }
     }

           stage('Push image in staging and deploy it') {
             when {
              expression { GIT_BRANCH == 'origin/master' }
            }
            agent {
              docker {image 'franela/dind'}
            }
             environment {
                 HEROKU_API_KEY = credentials('heroku_api_key')
             }  
           steps {
              script {
                sh '''
                    apk --no caher and npm
                    npm install -g heroku 
                    heroku container:login
                    heroku create $STAGING || echo "project already exist"
                    heroku container:push -a $PRODUCTION web
                    heroku container:release -a $PRODUCTION web
            '''
          }
        }
     }
       
       post {
         always {
           scripts {
             slackNotifier currentBuild.result
           }
         }
       }

     }
}
