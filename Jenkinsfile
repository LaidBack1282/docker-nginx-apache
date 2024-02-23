pipeline {
    agent {
        label 'worker'
    }

    stages {
        stage('Clone') {
            steps {
                dir('task17') {
                    script {
                        if (fileExists('.git')) {
                            sh 'git pull origin master'
                        } else {
                            sh 'git clone https://github.com/LaidBack1282/docker-nginx-apache.git .'
                        }
                    }
                }
            }
        }
    
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASSWORD'
                }
            }
        }
        
        stage('Docker Build Apache') {
            steps {
                dir('task17/apache') {
                    script {
                        env.APACHE_TAG = generateBuildTag()
                        sh """
                            docker build -t hika1282/apache:${env.APACHE_TAG} .
                            docker push hika1282/apache:${env.APACHE_TAG}
                            docker rmi -f hika1282/apache:${env.APACHE_TAG}
                        """
                    }
                }
            }
        }
        
        stage('Docker Build Nginx') {
            steps {
                dir('task17/nginx') {
                    script {
                        env.NGINX_TAG = generateBuildTag()
                        sh """
                            docker build -t hika1282/nginx:${env.NGINX_TAG} .
                            docker push hika1282/nginx:${env.NGINX_TAG}
                            docker rmi -f hika1282/nginx:${env.NGINX_TAG}
                        """
                    }
                }
            }
        }
        
        stage('Deploy Docker Compose to EC2 and Run') {
            steps {
                script {
                    env.EC2_IP = '3.79.247.105'
                    withCredentials([sshUserPrivateKey(credentialsId: 'ubuntu', keyFileVariable: 'SSH_KEY')]) {
                        sh """
                            pwd
                            scp -o StrictHostKeyChecking=no -i $SSH_KEY ./task17/docker-compose.yaml ${env.EC2_IP}:/home/ubuntu/
                            ssh -o StrictHostKeyChecking=no -i $SSH_KEY ${env.EC2_IP} "export APACHE_TAG=${env.APACHE_TAG}; export NGINX_TAG=${env.NGINX_TAG}; docker-compose -f /home/ubuntu/docker-compose.yaml up -d"
                        """
                    }
                }
            }
        }
    }
}

def generateBuildTag() {
    def date = new Date()
    def formattedDate = date.format('yyyyMMdd-HHmmss')
    return "v-${formattedDate}"
}
