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

    parameters {
        string(name: 'APP_BRANCH', defaultValue: 'main', description: 'Rama del microservicio')
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                git branch: "${params.APP_BRANCH}", url: 'https://github.com/roberto14118927/node-healthcheck.git'
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
                        def ip = params.APP_BRANCH == 'develop' ? DEV_IP :
                                params.APP_BRANCH == 'qa'      ? QA_IP :
                                params.APP_BRANCH == 'main'    ? PROD_IP : null
                        def pm2_name = "${params.APP_BRANCH}-health"

                        if (ip == null) {
                            error "Branch ${params.APP_BRANCH} no está configurada para despliegue."
                        }

                        sh """
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no $EC2_USER@$ip '
                            cd $REMOTE_PATH &&
                            git pull origin ${params.APP_BRANCH} &&
                            npm ci &&
                            pm2 restart ${pm2_name} || pm2 start server.js --name ${pm2_name}
                        '
                        """
                    }
                }
            }

    }
}
