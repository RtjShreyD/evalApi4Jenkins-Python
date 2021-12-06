Jenkinsfile (Declarative Pipeline)
    pipeline {
        agent any
        options {
            timeout(time: 30, unit: 'MINUTES')
            buildDiscarder(logRotator(numToKeepStr: '20'))
            disableConcurrentBuilds()
            timestamps()
        }
        triggers {
            pollSCM('* * * * *') 
        }
        parameters {
            string(
                name: 'GIT_BRANCH',
                defaultValue: 'master',
                description: 'The branch to be deployed'
            )
            
            
            booleanParam(
                name: 'DEBUG_MODE',
                defaultValue: false,
                description: 'Select if you want to Run in debug mode'
            )

        }
        environment {
			GIT_REPO = 'git@github.com:Stackexpress-Shreyans/testOpsnginx.git'
            REMOTE_SERVER_USER = 'root'
            REMOTE_SERVER_ADDRESS_1 = '10.10.10.11'
            REMOTE_SERVER_ADDRESS_2 = '10.10.10.12'
			APP_REGISTRY = 'registry.gitlab.com/stackexpresso/opspi2020/users-data-dev/NextTestOpsNginxJenkinsBuildRepo-4'
            RAILS_ENV = 'production'
        
        }
        stages {
            // stage('Start') {
            //     when {
            //         environment name: 'DEBUG_MODE', value: 'false'
            //     }
            //     steps {
            //         slackSend (color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            //     }
            // }
            stage('Update Code') {

                steps {
                    timestamps {
                        // Clone the repo
                        timeout(time: 300, unit: 'SECONDS') {
                            dir('app-code') {
                                git poll: true, credentialsId: 'opsbot-dev', branch: "$GIT_BRANCH", url: "$GIT_REPO"
                                sh 'git branch'
                                sh 'git status'
                                sh 'git log'
                            }
                        }
                    }
                }
            }
            stage('build') {
                steps {
                    timestamps {
                
                        timeout(time: 1200, unit: 'SECONDS') {
                            dir('app-code') {
                                // credentialid below will be diff for each repo created
                                withDockerRegistry(credentialsId: 'testops-deploy', url: 'https://registry.gitlab.com/'){ 
                            
                                
                                sh """#!/bin/bash
                                set -e
                                docker build --build-arg RAILS_ENV  -t $APP_REGISTRY:${env.BUILD_TAG} -t $APP_REGISTRY:latest .
                                docker push  $APP_REGISTRY:${env.BUILD_TAG}
                                docker push  $APP_REGISTRY:latest
                                """
                                
                                }
                                    
                                
                            
                            }
                        }
                    }
                }
            }
            
            stage('Remote Checks and deploy') {
                steps {
                    sshagent(['coralJenkinsPublic']) {
                        timestamps {
                            timeout(time: 300, unit: 'SECONDS') {
                                sh '''#!/bin/bash
                                set -e                            
                                ssh -o StrictHostKeyChecking=no $REMOTE_SERVER_USER@$REMOTE_SERVER_ADDRESS pwd
                                scp -r $WORKSPACE/app-code/docker-compose.yml  $REMOTE_SERVER_USER@$REMOTE_SERVER_ADDRESS:/srv/docker-apps/cfal-api 
                                ssh -o StrictHostKeyChecking=no $REMOTE_SERVER_USER@$REMOTE_SERVER_ADDRESS "bash -c ' cd /srv/docker-apps/cfal-api && APP_REGISTRY=registry.gitlab.com/stackexpress-revel/cfal-api/production:$BUILD_TAG  /usr/local/bin/docker-compose up -d '" 
                                
                                '''
                            }
                            
                        }
                    }
                }
            }
            stage('Remote Checks and deploy another server') {
                steps {
                    sshagent(['coralJenkinsPublic']) {
                        timestamps {
                            timeout(time: 300, unit: 'SECONDS') {
                                sh '''#!/bin/bash
                                set -e                            
                                ssh -o StrictHostKeyChecking=no $REMOTE_SERVER_USER@$ANOTHER_REMOTE_SERVER_ADDRESS pwd
                                scp -r $WORKSPACE/app-code/docker-compose.yml  $REMOTE_SERVER_USER@$ANOTHER_REMOTE_SERVER_ADDRESS:/srv/docker-apps/cfal-api 
                                ssh -o StrictHostKeyChecking=no $REMOTE_SERVER_USER@$ANOTHER_REMOTE_SERVER_ADDRESS "bash -c ' cd /srv/docker-apps/cfal-api && APP_REGISTRY=registry.gitlab.com/stackexpress-revel/cfal-api/production:$BUILD_TAG  /usr/local/bin/docker-compose up -d '" 
                                
                                '''
                            }
                            
                        }
                    }
                }
            }
            
            
        }
        post {
            always {
                echo 'Job complete'
            }
            success {
                echo 'Job successful'

                script {
                    
                    if (params.DEBUG_MODE == false) {
                        // slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
                        sh 'echo "Hi"'
                    }
                }

            }
            failure {
                echo 'Job failed'

                script {
                    if (params.DEBUG_MODE == false) {
                        //slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
                        sh 'echo "Hi Hi"'
                    }
                }
                
            }
            unstable {
                echo 'Job unstable'
            }
            changed {
                echo 'Job state changed'
            }
        }

    }