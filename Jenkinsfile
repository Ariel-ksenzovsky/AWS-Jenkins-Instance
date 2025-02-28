pipeline {
    agent any

    environment {
        SQL_FILE_PATH = '/tmp/init.sql'
        DB_PASSWORD = credentials('mysql_root')
        DOCKER_TOKEN_ID = 'docker-hub-token'
        DOCKER_IMAGE = 'arielk2511/star_meme_sql_compose'
        BUILD_NUM = "${BUILD_NUMBER}"
    }

    triggers {
        pollSCM('* * * * *')  // poll SCM every minute
    }

    stages {
        stage('Cleanup') {
            steps {
                sh '''
                docker stop docker-gif-app || echo "Container not running"
                docker rm docker-gif-app || echo "Container already removed"
                rm -rf ${WORKSPACE}/* || true
                git clone https://github.com/Ariel-ksenzovsky/AWS-Jenkins-Instance.git
                pwd
                docker compose down || true
                docker rmi $(docker images -q) -f || true
                '''
            }
        }

        stage('Set the Environment Variables') {
            steps {
                sh '''
                pwd
                id
                cp /home/arielk/.env /var/lib/jenkins/workspace/test1/AWS-Jenkins-Instance
                '''
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    withCredentials([string(credentialsId: env.DOCKER_TOKEN_ID, variable: 'DOCKER_TOKEN')]) {
                        sh """
                        echo "$DOCKER_TOKEN" | docker login -u arielk2511 --password-stdin
                        """
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    sh '''
                    cd AWS-Jenkins-Instance
                    docker build -t ${DOCKER_IMAGE}:latest .
                    docker build -t ${DOCKER_IMAGE}:2.0.${BUILD_NUM} .
                    docker push ${DOCKER_IMAGE}:latest
                    docker push ${DOCKER_IMAGE}:2.0.${BUILD_NUM}
                    '''
                }
            }
        }

        stage('Prepare SQL File') {
            steps {
                sh '''
                cp AWS-Jenkins-Instance/init.sql ${SQL_FILE_PATH}
                '''
            }
        }

        stage('Run') {
            steps {
                sh '''
                docker ps -a
                id
                pwd
                cd AWS-Jenkins-Instance
                pwd
                docker compose up -d
                docker ps
                '''
            }
        }

        stage('Test-web') {
            steps {
                sh 'sleep 30'
                sh '''
                if ! docker logs docker-gif-app; then
                    echo "Container logs check failed"
                    exit 1
                fi
                if ! curl -f http://localhost:5000; then
                    echo "App is not reachable."
                    docker logs docker-gif-app
                    exit 1
                fi
                '''
            }
        }

        stage('Deploy with Terraform') {
            steps {
                script {
                    // Run Terraform init and apply
                    sh '''
                    cd AWS-Jenkins-Instance-terraform
                    pwd
                    cd terraform
                    terraform init
                    terraform apply -auto-approve
                    '''
                }
            }
        }
    }


    post {
        always {
            sh '''
            rm -f ${SQL_FILE_PATH}
            '''
        }
    }
}