pipeline {
    agent any
 
    environment {
        GIT_REPO_NAME = "webapp"
        GIT_USER_NAME = "Hariteja234"
    }
 
    stages {
        stage('Code Checkout with checkout') {
            steps {
                checkout([$class: 'GitSCM',
                        branches: [[name: 'main']],
                        userRemoteConfigs: [[url: 'https://github.com/pradeep2025-source/webapp.git',
                        credentialsId: '5bf9bcf1-a7f6-4a2e-8859-c8f510a7cadd']]])
            }
        }
 
       /* stage('SonarQube Scan') {
            steps {
                // Navigate to the correct directory where pom.xml exists
                dir('my-webapp') {
                    sh '''
                        mvn sonar:sonar \\
                        -Dsonar.host.url=http://172.20.61.65:9000/ \\
                        -Dsonar.login=squ_37dbfb0046edf20083ecec00b7e45c7d2792e63b
                    '''
                }
            }
        }*/
 
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
                   sh 'docker build -t pradeepbrucelee/Frontend:${BUILD_NUMBER} .'
               }
            }
        }
 
        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerhub-token', variable: 'dockerhub_token')]) {
                        sh 'docker login -u pradeepbrucelee -p ${dockerhub_token}'
                        sh 'docker push pradeepbrucelee/Frontend:${BUILD_NUMBER}'
                        echo 'Docker Image Pushed to Docker Hub'
                    }
                 }
            }
        }
        stage('Update Deployment File') {
          environment {
            GIT_REPO_NAME = "webapp"
            GIT_USER_NAME = "pradeepbrucelee"
          }
          steps {
            echo 'Update Deployment File'
            withCredentials([string(credentialsId: '5bf9bcf1-a7f6-4a2e-8859-c8f510a7cadd', variable: '5bf9bcf1-a7f6-4a2e-8859-c8f510a7cadd')]) {
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
    }
}