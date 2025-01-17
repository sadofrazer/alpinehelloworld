pipeline{
    environment{
        IMAGE_NAME = "sadofrazer/alpinehelloworld"
        IMAGE_TAG = "latest"
        CONTAINER_NAME = "alpinehelloworld"
        STAGING = "ajc-staging"
        PRODUCTION = "ajc-production"
    }

    agent none

    stages{

        stage ('Build Image') {
            agent any
            steps{
                script{
                    sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} . '
                }
            }
        }

        stage ('Try to clean Container') {
            agent any
            steps{
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    script{
                        sh'''
                           docker stop ${CONTAINER_NAME}
                           docker rm ${CONTAINER_NAME}
                        '''
                    }
                }
            }
        }
   
        stage ('Run container based on builded image') {
            agent any
            steps{
                script{
                    sh'''
                       docker run -d --name ${CONTAINER_NAME} -e PORT=5000 -p 5000:5000 ${IMAGE_NAME}:${IMAGE_TAG}
                       sleep 5
                       curl http://172.17.0.1:5000 | grep -q "Hello world!"
                    '''
                }
            }
        }

        stage ('Test image') {
            agent any
            steps{
                script{
                    sh'''
                       curl http://172.17.0.1:5000 | grep -q "Hello world!"
                    '''
                }
            }
        }

        stage ('Clean Container') {
            agent any
            steps{
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    script{
                        sh'''
                           docker stop ${CONTAINER_NAME}
                           docker rm ${CONTAINER_NAME}
                        '''
                    }
                }
            }
        }

        stage ('push image and deploy it in staging env') {
            when{
                expression{ GIT_BRANCH == 'origin/master'}
            }
            agent any
            environment{
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps{
                script{
                    sh'''
                       heroku container:login
                       heroku create ${STAGING} || echo "OK"
                       #Permet à Heroku de rebuilder l'application et de la pusher sur heroku
                       heroku container:push -a ${STAGING} web
                       #Heroku déploiera l'application dans l'environnement de staging
                       heroku container:release -a ${STAGING} web
                    '''
                }
            }
        }

        stage ('push image and deploy it in Production env') {
            when{
                expression{ GIT_BRANCH == 'origin/master'}
            }
            agent any
            environment{
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps{
                script{
                    sh'''
                       heroku container:login
                       heroku create ${PRODUCTION} || echo "project already exist"
                       #Permet à Heroku de rebuilder l'application et de la pusher sur heroku
                       heroku container:push -a ${PRODUCTION} web
                       #Heroku déploiera l'application dans l'environnement de Production
                       heroku container:release -a ${PRODUCTION} web
                    '''
                }
            }
        }

    }

    post {
        success {
         slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        failure {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }   
    }
}
