pipeline {
    agent none

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "mymaven"
    }
    
    environment{
        BUILD_SERVER_IP='ec2-user@172.31.91.225'
        IMAGE_NAME='rohanrace/private_1'
    }

    stages {
        stage('Compile') {
            // agent {label "linux_slave"}
            agent any
            steps {              
              script{
                     echo "COMPILING"
                     sh "mvn compile"
              }             
            }
            
        }
        stage('Test') {
            agent any
            steps {           
              script{
                   echo "RUNNING THE TC"
                   sh "mvn test"
                }              
             
            }            
        
        post{
            always{
                junit 'target/surefire-reports/*.xml'
            }
        }
        }
        stage('Containerise the app') {
            agent any
            steps {              

                script{
                     echo "Creating the docker image and Push to registry"
                sshagent(['slave2']) {
                withCredentials([usernamePassword(credentialsId: 'Docker_hub', passwordVariable: 'password', usernameVariable: 'rohanrace')]) {
                sh "scp -o StrictHostKeyChecking=no server-script.sh ${BUILD_SERVER_IP}:/home/ec2-user"
                sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER_IP} 'bash server-script.sh ${IMAGE_NAME} ${BUILD_NUMBER}'"
                sh "ssh ${BUILD_SERVER_IP} sudo docker login -u ${rohanrace} -p ${password}"
                sh "ssh ${BUILD_SERVER_IP} sudo docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                }
                }             
                }
            }            
        }
    }
}
