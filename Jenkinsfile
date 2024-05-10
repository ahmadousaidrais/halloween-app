pipeline {
    agent any

    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'main', 
        description: 'Git branch name')
        string(name: 'DOCKERHUB_USERNAME', defaultValue: '', 
        description: 'Docker Hub Username')
        string(name: 'HOST_PORT', defaultValue: '', 
        description: 'ENTER THE Host Port')
        string(name: 'CONTAINER_NAME', defaultValue: 'my-container', 
        description: 'ENTER THE Container Name')
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    git branch: "${params.GIT_BRANCH}", credentialsId: 'github-catchup', 
                    url: 'https://github.com/chrisdylan237/halloween-app.git'
                }
            }
        }

        stage('Build SonarQube Scanner CLI Image') {
            steps {
                script {
                    dir("${WORKSPACE}/sonar") {
                        sh "docker build -t ${params.DOCKERHUB_USERNAME}/sonar-scanner-cli:${BUILD_NUMBER} ."
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv("SonarScanner") {
                        docker.image("${params.DOCKERHUB_USERNAME}/sonar-scanner-cli:${BUILD_NUMBER}").inside {
                            sh "sonar-scanner"
                        }
                    }
                }
            }
        }

        stage('Build Application') {
            steps {
                script {
                    sh "docker build -t ${params.DOCKERHUB_USERNAME}/catchup:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-dylan', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh "docker login -u ${USERNAME} -p ${PASSWORD}"
                    }
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                script {
                    sh "docker push ${params.DOCKERHUB_USERNAME}/catchup:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    sh "docker run -itd -p ${params.HOST_PORT}:80 --name ${params.CONTAINER_NAME} ${params.DOCKERHUB_USERNAME}/catchup:${BUILD_NUMBER}"
                    sh "docker ps | grep ${params.CONTAINER_NAME}"
                }
            }
        }
    }
}
