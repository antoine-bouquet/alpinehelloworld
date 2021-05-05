pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "antoine-staging"
        PRODUCTION = "antoine-production"
        IMAGE_REPO = "antoinebouquet1010"
    }
    agent none
    stages {
        stage('Build Image') {
            agent any
            steps {
                script {
                    echo 'Building..'
                    sh 'docker build -t ${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    docker run --name ${IMAGE_NAME} -d -p 80:5000 -e PORT=5000 ${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
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
                        curl http://172.17.0.1 | grep -q "Hello world"
                    '''
              }
           }
        }
        stage('Cleaning Container') {
            agent any
            steps {
                script {
                    sh 'docker rm -vf ${IMAGE_NAME}'
                }
            }
        }
	stage('Push image on dockerhub') {
           agent any 
           environment {
                DOCKERHUB_LOGIN = credentials('ded4d165-8a67-471f-8797-a6a89d48abc9')
            }
           steps {
               script {
                   sh '''
		   docker login --username ${DOCKERHUB_LOGIN_USR} --password ${DOCKERHUB_LOGIN_PSW}
                   docker push ${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                   '''
               }
           }
        }
        stage('Push image in staging and deploy it') {
            when {
              expression { GIT_BRANCH == 'origin/master' }
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
                script {
                    sh '''
                    heroku container:login
                    heroku create $STAGING || echo "project already exist"
                    heroku container:push -a $STAGING web
                    heroku container:release -a $STAGING web
                    '''
                }
            }
        }
    	stage('Test Staging deployment') {
       when {
              expression { GIT_BRANCH == 'origin/master' }
            }
           agent any
           steps {
              script {
                sh '''
                    curl https://${STAGING}.herokuapp.com | grep -q "Hello world"
                '''
              }
           }
      }
        stage('Push image in production and deploy it') {
            when {
              expression { GIT_BRANCH == 'origin/master' }
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
                script {
                    sh '''
                    heroku container:login
                    heroku create $PRODUCTION || echo "project already exist"
                    heroku container:push -a $PRODUCTION web
                    heroku container:release -a $PRODUCTION web
                    '''
                }
            }
        }
        stage('Test Prod deployment') {
       when {
              expression { GIT_BRANCH == 'origin/master' }
            }
           agent any
           steps {
              script {
                sh '''
                    curl https://${PRODUCTION}.herokuapp.com | grep -q "Hello world"
                '''
              }
           }
      }
    }
}
