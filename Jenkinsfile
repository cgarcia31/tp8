pipeline {
     environment {
       ID_DOCKER = "${ID_DOCKER_PARAMS}"
       IMAGE_NAME = "webapp"
       IMAGE_TAG = "latest"
       STAGING = "${ID_DOCKER}-staging"
       PRODUCTION = "${ID_DOCKER}-production"
     }

     agent none
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
                  sh '''
		  	                echo "Build du container"
 			                docker build -t ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG} .
		              '''
                }
             }
        }

        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
		 	              echo "Run du conteneur"
			              docker rm -f ${IMAGE_NAME}
		 	              docker run -d --name ${IMAGE_NAME} -p ${PORT_EXPOSED}:5000 -e PORT=5000 ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG} 
		              '''
               }
            }
       }

       stage('Test image') {
           agent any
           steps {
              script {
                sh 'curl -I http://${DOCKER_IP}:${PORT_EXPOSED} | grep -q "200"'
              }
           }
      }
	stage('test imagewith trivy') {
		agent any
		steps {
			sh "trivy  image --severity CRITICAL -f json -o results-image.json ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}"
			recordIssues(tools: [trivy(pattern: '*.json')])
			sh 'cat result-image.json | grep -q Vulnerabilities'
			}
	}
	/*
	stage('test security trivy'){
		agent any
               steps {
                  sh "trivy fs -f json -o results-local.json --security-checks vuln,secret,config ./"
                  recordIssues(tools: [trivy(pattern: 'results.json')])
  		}
	}
	stage('test webapp repo'){
		agent any
		steps {
			sh "trivy -f json -o results.json repo https://github.com/diranetafen/static-website-example.git"
                  	recordIssues(tools: [trivy(pattern: 'results.json')])
		}
	}
	*/

      stage('Clean Container') {
          agent any
          steps {
             script {
               sh '''
	       	        echo "Clean du container"
	       	        docker stop ${IMAGE_NAME} && docker rm ${IMAGE_NAME}
          	     '''
              }
          }
      }

     stage ('Login and Push Image on docker hub') {
          agent any
          environment{
            DOCKERHUB_PASSWORD=credentials("${DOCKERHUB_PASSWORD_CREDID}")
          }
          steps {
             script {
               sh '''
			          echo $DOCKERHUB_PASSWORD_PSW | docker login -u $ID_DOCKER --password-stdin
			          docker push ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
               '''
             }
          }
      }
     
     stage('Push image in staging and deploy it') {
       when {
              expression { GIT_BRANCH == 'origin/main' }
            }
      agent any
      environment{
        tempo=credentials("${HEROKU_API_CREDID}")
        HEROKU_API_KEY="${tempo_PSW}"
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

     stage('Push image in production and deploy it') {
       when {
              expression { GIT_BRANCH == 'origin/main' }
            }
      agent any
      environment{
        tempo=credentials("${HEROKU_API_CREDID}")
        HEROKU_API_KEY="${tempo_PSW}"
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
  }
}
