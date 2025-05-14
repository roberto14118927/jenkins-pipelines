pipeline {
    agent any

    environment {
        EC2_USER = 'ubuntu'
        SSH_KEY = credentials('ssh-key-ec2')
        DEV_IP = '11.11.11.11'
        QA_IP  = '22.22.22.22'
        PROD_IP = '98.81.245.108'
        REMOTE_PATH = '/home/ubuntu/node-healthcheck'
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                script {
                    def branchName = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    env.APP_BRANCH = branchName
                    git branch: "${env.APP_BRANCH}", url: 'https://github.com/roberto14118927/node-healthcheck.git'
                }
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
                    def ip = env.APP_BRANCH == 'develop' ? DEV_IP :
                             env.APP_BRANCH == 'qa'      ? QA_IP :
                             env.APP_BRANCH == 'main'    ? PROD_IP : null
                    def pm2_name = "${env.APP_BRANCH}-health"

                    if (ip == null) {
                        error "Branch ${env.APP_BRANCH} no está configurada para despliegue."
                    }

                    sh """#!/bin/bash
                    ssh -i $SSH_KEY -o StrictHostKeyChecking=no $EC2_USER@$ip << 'ENDSSH'
                        cd $REMOTE_PATH &&
                        git pull origin ${env.APP_BRANCH} &&
                        npm ci &&
                        pm2 restart ${pm2_name} || pm2 start server.js --name ${pm2_name}
                    ENDSSH
                    """
                }
            }
        }
    }
}
