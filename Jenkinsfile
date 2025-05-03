pipeline {
    agent any
 
    stages {
        stage('Code Checkout with checkout') {
            steps {
                checkout([$class: 'GitSCM',
                        branches: [[name: 'main']],
                        userRemoteConfigs: [[url: 'https://github.com/pradeep2025-source/webapp.git',
                        credentialsId: '5bf9bcf1-a7f6-4a2e-8859-c8f510a7cadd']]])
            }
        }
 
        stage('Build Artifact') {
            steps {
                // Navigate to the correct directory where pom.xml exists
                dir('my-webapp') {
                    sh 'mvn clean package'
                }
            }
        }
 
       stage('Build Docker Image') {
           steps {
               dir('my-webapp') {
                   sh 'sudo docker build -t pradeepbrucelee/frontend:${BUILD_NUMBER} .'
               }
            }
        }
 
        stage('Push to Docker Hub') {
            steps {
                script {

                        sh 'sudo docker login -u pradeepbrucelee -p Chmode@123'
                        sh 'sudo docker push pradeepbrucelee/frontend:${BUILD_NUMBER}'
                        echo 'Docker Image Pushed to Docker Hub'
                  /*  withCredentials([string(credentialsId: 'dockerhub-token', variable: 'dockerhub-token')]) {
                        sh 'sudo docker login -u pradeepbrucelee -p ${dockerhub-token}'
                        sh 'sudo docker push pradeepbrucelee/frontend:${BUILD_NUMBER}'
                        echo 'Docker Image Pushed to Docker Hub'
                    }*/
                 }
            }
        }
        stage('Update Deployment File') {
          environment {
            GIT_REPO_NAME = "webapp"
            GIT_USER_NAME = "pradeep2025-source"
          }
          steps {
            echo 'Update Deployment File'
            withCredentials([string(credentialsId: 'github', variable: 'github')]) {
              sh """
                # Configure Git
                # git config user.email "harikumar.cloud@gmail.com"
                # git config user.name "hari" 
 
                # Update the image tag in deployment YAML
                sed -i "s|Frontend:.*|Frontend:${BUILD_NUMBER}|g" my-webapp/deployment.yml
 
                # Commit and push
                git add my-webapp/deployment.yml
                git commit -m "Update deployment image to version ${BUILD_NUMBER}" || echo "No changes to commit"
                git push https://${github}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
                 """
             }
             } 
        }
        
        stage('Deploy via SSH') {
            steps {
                    sh '''
                        ssh -i /home/ubuntu/Docker.pem  ubuntu@172.31.20.74   'kubctl apply -f /var/lib/jenkins/workspace/k8s/my-webapp/deployment.yml'

                    '''
                }
        } 

    }
}