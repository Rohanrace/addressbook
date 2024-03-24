pipeline {
    agent none
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "mymaven"
    }
    parameters{
        string(name:'Env',defaultValue:'Test',description:'version to deploy')
        booleanParam(name:'executeTests',defaultValue: true,description:'decide to run tc')
        choice(name:'APPVERSION',choices:['1.1','1.2','1.3'])
    }

    stages {
        stage('Compile') {
            agent {label 'linux_slave'}
            steps {
               echo "compiling the code ${params.APPVERSION}"
               sh 'mvn compile'
            }
        }
        stage('UnitTest') {
            agent any
            when{
                expression{
                    params.executeTests == true
                }
            }
            steps {
                script{
               echo "Test the code"
               sh 'mvn test'
            }
            }
        }
        stage('Package') {
            agent any
            steps {
                script{
               echo "Package the code ${params.Env}"
               sh 'mvn package'
            }
            }
        }
        }
        stage('DEPLOY') {
            input{
                message "Select the PLATFFORM to deploy"
                ok "PLATFORM Selected"
                parameters{
                    choice(name:'PLATFORM',choices:['EKS','ONPREM_K8s','SERVERS'])
                }
            }
            steps {
               echo "DEPLOY the code ${params.Env}"
            }
        }
    }
