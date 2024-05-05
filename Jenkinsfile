pipeline {
    agent none
    // tools{
    //     jdk 'myjava'
    //     maven 'mymaven'
    // }
     environment{
        IMAGE_NAME='devopstrainer/java-mvn-privaterepos'
        DEV_SERVER_IP='rohan@172.31.0.193'
        APP_NAME='java-mvn-app'
    }
    stages {
        // stage('COMPILE') {
        //     agent any
        //     steps {
        //         script{
        //             echo "COMPILING THE CODE"
        //             git 'https://github.com/preethid/addressbook.git'
        //             sh 'mvn compile'
        //         }
        //                   }
        //     }
        // stage('UNITTEST'){
        //     agent any
        //     steps {
        //         script{
        //             echo "RUNNING THE UNIT TEST CASES"
        //             sh 'mvn test'
        //         }
              
        //     }
        //     post{
        //         always{
        //             junit 'target/surefire-reports/*.xml'
        //         }
        //     }
        //     }
        stage('PACKAGE+BUILD DOCKER IMAGE ON BUILD SERVER'){
            agent any
           steps{
            script{
            sshagent(['slave-tf2']) {
        withCredentials([usernamePassword(credentialsId: 'Docker_hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                     echo "PACKAGING THE CODE"
                     sh "scp -rvvv server-script.sh ${DEV_SERVER_IP}:/home/rohan"
                     sh "ssh -o StrictHostKeyChecking=no ${DEV_SERVER_IP} 'bash ~/server-script.sh ${IMAGE_NAME} ${BUILD_NUMBER}'"
                    //sh "ssh ${DEV_SERVER_IP} sudo docker build -t  ${IMAGE_NAME} /home/rohan/addressbook"
                    sh "ssh ${DEV_SERVER_IP} sudo docker login -u $USERNAME -p $PASSWORD"
                    sh "ssh ${DEV_SERVER_IP} sudo docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                    }
                    }
                }
            }
        }
        stage("Provision deploy server with TF"){
            environment{
                   AWS_ACCESS_KEY_ID =credentials("jenkins_aws_access_key_id")
                   AWS_SECRET_ACCESS_KEY=credentials("jenkins_aws_secret_access_key")
            }
             agent any
                   steps{
                       script{
                           dir('terraform'){
                           sh "terraform init"
                           sh "terraform apply --auto-approve"
                           EC2_PUBLIC_IP = sh(
                            script: "terraform output ec2-ip",
                            returnStdout: true
                           ).trim()
                       }
                       }
                   }
        }
        stage('DEPLOY ON EC2 instance'){
            agent any
                steps{
                    script{
            echo "RUN THE APP ON ec2 instance"
               echo "Waiting for ec2 instance to initialise"
               sleep(time: 90, unit: "SECONDS")
               echo "Deploying the app to ec2-instance provisioned bt TF"
               echo "${EC2_PUBLIC_IP}"
               sshagent(['slave-tf2']) {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                      sh "ssh -o StrictHostKeyChecking=no rohan@${EC2_PUBLIC_IP} sudo docker login -u $USERNAME -p $PASSWORD"
                      sh "ssh rohan@${EC2_PUBLIC_IP} sudo docker run -itd -p 8080:8080 ${IMAGE_NAME}:${BUILD_NUMBER}"
                      
                }
            }
            }
                }}    
    }
}
