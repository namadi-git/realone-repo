pipeline {
    agent any
    environment {
        VERSION = "${env.BUILD_ID}"
        AWS_ACCOUNT_ID="636885593592"
        AWS_DEFAULT_REGION="us-east-2"
        IMAGE_REPO_NAME="my_app_ecr"
        IMAGE_TAG= "${env.BUILD_ID}"
        REPOSITORY_URI = "636885593592.dkr.ecr.us-east-2.amazonaws.com/my_app_ecr"
    }
    stages {
        stage('Git checkout') {
            steps {
                git 'https://github.com/namadi-git/realone-repo.git'
            }
        }
        
        stage('Build with maven') {
            steps {
                sh 'cd SampleWebApp && mvn clean install'
            }
        }
        
             stage('Test') {
            steps {
                sh 'cd SampleWebApp && mvn test'
            }
        
            }
        
        
         stage('Logging into AWS ECR') {
                     environment {
                        AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
                        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_key')
                         
                   }
                     steps {
                       script{
             
                         sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
                 
            }
        }
      
          stage('Building image') {
            steps{
              script {
                dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
    }
        
        stage('Pushing to ECR') {
          steps{  
            script {
                sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:${IMAGE_TAG}"
                sh "docker push ${REPOSITORY_URI}:${IMAGE_TAG}"
                sh "docker tag ${REPOSITORY_URI}:${IMAGE_TAG} ${REPOSITORY_URI}:latest"
                sh "docker push ${REPOSITORY_URI}:latest"
         }
        }
      }
         stage('Remove Unused docker image') {
          steps{
              sh "docker rmi ${REPOSITORY_URI}:${IMAGE_TAG}"
              sh "docker rmi ${REPOSITORY_URI}:latest"
                        }
            }  
    
         stage('pull image & Deploying application on eks cluster') {
                    environment {
                       AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
                       AWS_SECRET_ACCESS_KEY = credentials('aws_secret_key')
                 }
                    steps {
                      script{
                        dir('kubernetes/') {
                          sh 'aws eks update-kubeconfig --name myapp-eks-cluster --region us-east-2'
                          sh 'aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com'
                          sh 'helm upgrade --install --set image.repository="${REPOSITORY_URI}" --set image.tag= "${VERSION}" myjavaapp myapp/ ' 



 
                        }
                    }
               }
            }
    }
}
