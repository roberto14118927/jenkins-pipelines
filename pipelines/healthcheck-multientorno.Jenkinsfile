// pipelines/healthcheck-multientorno.Jenkinsfile

pipeline {
    agent any

    environment {
        EC2_USER = 'ubuntu'
        SSH_KEY = credentials('ssh-key-ec2')
        DEV_IP = '11.11.11.11'
        QA_IP  = '22.22.22.22'
        PROD_IP = '33.33.33.33'
        REMOTE_PATH = '/home/ubuntu/node-healthcheck'
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                // Clona el repo real con el microservicio
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/tu-org/node-healthcheck.git'
            }
        }

        stage('Build') {
            steps {
                sh 'rm -rf node_modules'
                sh 'npm ci'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def ip = BRANCH_NAME == 'develop' ? DEV_IP :
                             BRANCH_NAME == 'qa'      ? QA_IP :
                             BRANCH_NAME == 'main'    ? PROD_IP : null
                    def pm2_name = BRANCH_NAME + '-health'

                    if (ip == null) {
                        error "Branch ${BRANCH_NAME} no está configurada para despliegue."
                    }

                    sh """#!/bin/bash
                    ssh -i $SSH_KEY -o StrictHostKeyChecking=no $EC2_USER@$ip << 'ENDSSH'
                        cd $REMOTE_PATH &&
                        git pull origin ${BRANCH_NAME} &&
                        npm ci &&
                        pm2 restart ${pm2_name} || pm2 start server.js --name ${pm2_name}
                    ENDSSH
                    """
                }
            }
        }
    }
}
