pipeline {
    agent { label 'slave1' }

    stages {
        stage('checkout') {
            steps {
                sh 'rm -rf hello-world-war'
                sh 'git clone https://github.com/Chaitraradha/hello-world-war.git'
            }
        }
        
        stage('build') {
            steps {
                dir("hello-world-war") {
                    sh 'echo "inside build"'
                    sh "docker build -t tomcat-war:${BUILD_NUMBER} ."
                }
            }
        }
        
        stage('push') {
            steps {
                withCredentials([usernamePassword(credentialsId: '9edb749c-52c9-40d2-9266-024789f72979', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                    sh "docker tag tomcat-war:${BUILD_NUMBER} chaitra0105/docker_tomcat:${BUILD_NUMBER}"
                    sh "docker push chaitra0105/docker_tomcat:${BUILD_NUMBER}"
                }
            }
        }
        
        stage('deploy') {
            parallel {
                stage('deployQA') {
                    agent { label 'slave2' }
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: '9edb749c-52c9-40d2-9266-024789f72979', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                                sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                                sh "docker pull chaitra0105/docker_tomcat:${BUILD_NUMBER}"
                                sh 'docker rm -f tomcat-qa || true'
                                sh 'docker run -d -p 5555:8080 --name tomcat-qa chaitra0105/docker_tomcat:${BUILD_NUMBER}'
                            }
                        }
                    }
                }
                stage('deployProd') {
                    agent { label 'slave3' }
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: '9edb749c-52c9-40d2-9266-024789f72979', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                                sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                                sh "docker pull chaitra0105/docker_tomcat:${BUILD_NUMBER}"
                                sh 'docker rm -f tomcat-prod || true'
                                sh 'docker run -d -p 8888:8080 --name tomcat-prod chaitra0105/docker_tomcat:${BUILD_NUMBER}'
                            }
                        }
                    }
                }
            }
        }
    }
}
