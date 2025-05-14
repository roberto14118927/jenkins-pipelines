pipeline {
    agent any

     tools {
        nodejs 'NodeJS'  
    }

    environment {
        EC2_USER = 'ubuntu'
        SSH_KEY = credentials('ssh-key-ec2')
        DEV_IP = '3.92.207.25'
        QA_IP  = '22.22.22.22'
        PROD_IP = '98.81.245.108'
        REMOTE_PATH = '/home/ubuntu/node-healthcheck'
    }

    stages {
        stage('Detect Branch') {
            steps {
                script {
                    env.ACTUAL_BRANCH = env.BRANCH_NAME ?: 'main'
                    echo "🔍 Rama activa: ${env.ACTUAL_BRANCH}"
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
                        def ip = params.APP_BRANCH == 'develop' ? DEV_IP :
                                params.APP_BRANCH == 'qa'      ? QA_IP :
                                params.APP_BRANCH == 'main'    ? PROD_IP : null

                        def pm2_name = "${params.APP_BRANCH}-health"
                        def home_path = REMOTE_PATH

                        if (ip == null) {
                            error "Branch ${params.APP_BRANCH} no está configurada para despliegue."
                        }

                         sh """
                            ssh -i $SSH_KEY -o StrictHostKeyChecking=no $EC2_USER@$ip '

                                echo "Actualizando sistema..."
                                sudo apt-get update -y &&
                                sudo apt-get upgrade -y

                                echo "Verificando Node.js..."
                                if ! command -v node > /dev/null; then
                                    curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
                                    sudo apt-get install -y nodejs
                                fi

                                echo "Verificando PM2..."
                                if ! command -v pm2 > /dev/null; then
                                    sudo npm install -g pm2
                                fi

                                echo "Verificando carpeta de app..."
                                if [ ! -d "$home_path/.git" ]; then
                                    git clone https://github.com/roberto14118927/node-healthcheck.git $home_path
                                fi

                                echo "Haciendo pull y deploy..."
                                cd $home_path &&
                                git pull origin ${params.APP_BRANCH} &&
                                npm ci &&
                                pm2 restart ${pm2_name} || pm2 start server.js --name ${pm2_name}
                            '
                            """

                        // sh """
                        // ssh -i $SSH_KEY -o StrictHostKeyChecking=no $EC2_USER@$ip '
                        //     cd $REMOTE_PATH &&
                        //     git pull origin ${params.APP_BRANCH} &&
                        //     npm ci &&
                        //     pm2 restart ${pm2_name} || pm2 start server.js --name ${pm2_name}
                        // '
                        // """

                        
                    }
                }
            }

    }
}
