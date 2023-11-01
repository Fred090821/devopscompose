pipeline {
    agent any
   options{
        buildDiscarder(logRotator(numToKeepStr:'5', daysToKeepStr:'5'))
   }

    environment{
       HUB_REGISTRY_ID = 'adedo2009'
       APP_IMAGE_NAME = 'devops-rest'
       TEST_IMAGE_NAME = 'devops-rest-test'
       DOCKERHUB_CREDENTIALS = credentials('docker-hub')
       COMPOSE_YML_PATH = '/Users/jaydenassi/.jenkins/workspace/devopscompose'
       dockerImage = ''
   }

    stages {
        stage(' Verify Tooling ') {
            steps {
            echo '=== Verify Tooling ==='
                script {
                    try{
                        if (checkOs() == 'Windows') {
                            bat '''
                               docker version
                               docker info
                               python3 --version
                            '''
                        } else {
                            sh '''
                              docker version
                              docker info
                              python3 --version
                            '''
                        }
                    }catch(Exception e){
                        echo 'Exception Running Back End Server'
                        error('Aborting The Build')
                    }
                }
            }
        }
         stage(' Checkout ') {
            steps {
                echo '=== Checkout Devops Code ==='
                script {
                    // Clean the workspace
                    cleanWs()
                    properties([pipelineTriggers([pollSCM('*/30 * * * *')])])
                    checkout([$class: 'GitSCM', branches: [[name: 'development']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/Fred090821/devopscompose.git']]])
                }
            }
        }

        stage(' Start Rest Server...') {
            steps {
            echo '=== Start Back End Server ==='
                script {
                    try{
                        if (checkOs() == 'Windows') {
                            bat 'start/min /usr/bin/python3 ./python/rest_app.py'
                        } else {
                            sh 'nohup /usr/bin/python3 ./python/rest_app.py &'
                        }
                    }catch(Exception e){
                        echo 'Exception Running Back End Server'
                        error('Aborting The Build')
                    }
                }
            }
        }
        stage(' Run Rest Tests ') {
            steps {
            echo '=== Run Back End Tests ==='
                script {
                    try{
                        if (checkOs() == 'Windows') {
                            bat '/usr/local/bin/pytest ./python/backend_testing.py'
                        } else {
                            sh '/usr/local/bin/pytest ./python/backend_testing.py'
                        }
                    }catch(Exception e){
                        echo 'Exception Running Back End Test'
                        error('Aborting The Build')
                    }
                }
            }
        }
        stage(' Clean Environment ') {
            steps {
            echo '=== Clean Environment After Tests ==='
                script {
                    try{
                        if (checkOs() == 'Windows') {
                            bat '''
                               /usr/bin/python3 clean_environment.py
                             '''
                        } else {
                             sh '/usr/bin/python3 clean_environment.py'
                        }
                    }catch(Exception e){
                        echo 'Exception Cleaning The Environment'
                        error('Aborting The Build')
                    }
                }
            }
        }
        stage(' Docker Build Rest Image ') {
            steps {
                script {
                    try{
                        if (checkOs() == 'Windows') {
                           bat 'docker build -t $APP_IMAGE_NAME ./python'
                           bat 'docker build -t $TEST_IMAGE_NAME ./python'
                        } else {
                            sh 'docker build -t $APP_IMAGE_NAME ./python'
                            sh 'docker build -t $TEST_IMAGE_NAME ./python'
                        }
                    }catch(Exception e){
                        echo 'Exception Running Docker Build'
                        error('Aborting the build')
                    }
                }
            }
        }
        stage(' Log In To Docker hub ') {
            steps {
            echo '=== Log In To Docker hub ==='
                script {
                    try{
                        if (checkOs() == 'Windows') {
                            bat 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                        } else {
                            sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                        }
                    }catch(Exception e){
                        echo 'Exception Login into Ducker Hub'
                        error('Aborting The Build')
                    }
                }
            }
        }
        stage(' Tag & Push Rest Image ') {
            steps {
            echo '=== Tag & Push Rest Image ==='
                script {
                    try{
                        if (checkOs() == 'Windows') {
                            bat '''
                              docker tag $APP_IMAGE_NAME $HUB_REGISTRY_ID/$APP_IMAGE_NAME:latest
                              docker tag $APP_IMAGE_NAME $HUB_REGISTRY_ID/$APP_IMAGE_NAME:${BUILD_NUMBER}
                              docker push -a $HUB_REGISTRY_ID/$APP_IMAGE_NAME

                              docker tag $TEST_IMAGE_NAME $HUB_REGISTRY_ID/$TEST_IMAGE_NAME:latest
                              docker tag $TEST_IMAGE_NAME $HUB_REGISTRY_ID/$TEST_IMAGE_NAME:${BUILD_NUMBER}
                              docker push -a $HUB_REGISTRY_ID/$TEST_IMAGE_NAME
                            '''
                        } else {
                            sh '''
                              docker tag $APP_IMAGE_NAME $HUB_REGISTRY_ID/$APP_IMAGE_NAME:latest
                              docker tag $APP_IMAGE_NAME $HUB_REGISTRY_ID/$APP_IMAGE_NAME:${BUILD_NUMBER}
                              docker push -a $HUB_REGISTRY_ID/$APP_IMAGE_NAME

                              docker tag $TEST_IMAGE_NAME $HUB_REGISTRY_ID/$TEST_IMAGE_NAME:latest
                              docker tag $TEST_IMAGE_NAME $HUB_REGISTRY_ID/$TEST_IMAGE_NAME:${BUILD_NUMBER}
                              docker push -a $HUB_REGISTRY_ID/$TEST_IMAGE_NAME
                            '''
                        }
                    }catch(Exception e){
                        echo 'Exception Pushing Docker Build'
                        error('Aborting the build')
                    }
                }
            }
        }
         stage(' Set Image Version ') {
            steps {
            echo '=== Set Image Version BUILD_VERSION : ${BUILD_NUMBER}==='
                script {
                     try{
                        if (checkOs() == 'Windows') {
                            bat 'echo IMAGE_TAG=$BUILD_NUMBER > $COMPOSE_YML_PATH/.env'
                        } else {
                            sh 'echo IMAGE_TAG=$BUILD_NUMBER > $COMPOSE_YML_PATH/.env'
                        }
                    }catch(Exception e){
                        echo 'Exception Set Image Version'
                        error('Aborting The Build')
                    }
                }
            }
        }
         stage(' Start Docker compose ') {
            steps {
               echo 'Start containers using docker compose ===> '
               script {
                    try{
                        if (checkOs() == 'Windows') {
                            bat '''
                                docker compose -f $COMPOSE_YML_PATH/docker-compose.yml up -d --wait
                                docker compose -f $COMPOSE_YML_PATH/docker-compose.yml ps'''
                        } else {
                            sh '''
                                docker compose -f $COMPOSE_YML_PATH/docker-compose.yml up -d --wait
                                docker compose -f $COMPOSE_YML_PATH/docker-compose.yml ps'''

                        }
                    }catch(Exception e){
                        echo 'Exception docker compose starting container'
                        error('Aborting the build')
                    }
                }
            }
        }
        stage(' Docker Rest Tests ') {
            steps {
            echo '=== Run Back End Tests ==='
                script {
                    try{
                        if (checkOs() == 'Windows') {
                            bat '/usr/local/bin/pytest ./pythondockertest/docker_tests.py'
                        } else {
                            sh '/usr/local/bin/pytest ./pythondockertest/docker_tests.py'
                        }
                    }catch(Exception e){
                        echo 'Exception Docker Rest Tests'
                        error('Aborting The Build')
                    }
                }
            }
        }
    }
    post {
        always {
        echo '=== post Clean Environment ==='
            script {
                try{
                    if (checkOs() == 'Windows') {
                        bat '''
                              /usr/bin/python3 clean_environment.py
                              docker-compose down
                              docker rmi $APP_IMAGE_NAME
                              docker rmi $HUB_REGISTRY_ID/$APP_IMAGE_NAME
                              docker rmi $HUB_REGISTRY_ID/$APP_IMAGE_NAME:${BUILD_NUMBER}
                              docker rmi $TEST_IMAGE_NAME
                              docker rmi $HUB_REGISTRY_ID/$TEST_IMAGE_NAME
                              docker rmi $HUB_REGISTRY_ID/$TEST_IMAGE_NAME:${BUILD_NUMBER}
                        '''
                        def currentDirectory = pwd()
                           echo "CURRENT DIRECTORY: ${currentDirectory}"
                        // Clean the workspace
                        cleanWs()
                    } else {
                        sh '''
                              /usr/bin/python3 clean_environment.py
                              docker-compose down
                              docker rmi $APP_IMAGE_NAME
                              docker rmi $HUB_REGISTRY_ID/$APP_IMAGE_NAME
                              docker rmi $HUB_REGISTRY_ID/$APP_IMAGE_NAME:${BUILD_NUMBER}
                              docker rmi $TEST_IMAGE_NAME
                              docker rmi $HUB_REGISTRY_ID/$TEST_IMAGE_NAME
                              docker rmi $HUB_REGISTRY_ID/$TEST_IMAGE_NAME:${BUILD_NUMBER}
                        '''
                        def currentDirectory = pwd()
                           echo "CURRENT DIRECTORY: ${currentDirectory}"
                        // Clean the workspace
                        cleanWs()
                    }
                }catch(Exception e){
                        // Catch any exception and print details
                        echo "Exception post Clean Environment :::: ${e}"
                        error('Aborting the build')
                }
            }
        }
        success {
            echo 'All test run successfully'
        }
        unstable {
            echo 'The build is unstable'
        }
        changed {
            echo 'The pipeline  state has changed'
        }
    }
}

def checkOs(){
    if (isUnix()) {
        def uname = sh script: 'uname', returnStdout: true
        if (uname.startsWith("Darwin")) {
            return "Macos"
        }
        else {
            return "Linux"
        }
    }
    else {
        return "Windows"
    }
}

// Function to check if a directory exists
def fileExists(path) {
    def file = file(path)
    return file.exists() && file.isDirectory()
}

