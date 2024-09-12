pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('DockerHub-Cred')
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/s7rosine/ci-cd-project.git'
            }
        }

        stage('Testing') {
            agent {
                docker { image 'maven:3.8.5-openjdk-18' }
            }
            steps {
                sh '''
                pwd
                mvn clean 
                mvn test  
                '''
            }
        }

        stage('SonarQube Analysis') {
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli:5.0.1'
                }
            }
            environment {
                CI = 'true'
                scannerHome = '/opt/sonar-scanner'
            }
            steps {
                withSonarQubeEnv('Sonar') {
                    sh '''
                    ${scannerHome}/bin/sonar-scanner
                    '''
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t rosinebelle/cicdproject:${BUILD_NUMBER} -f Dockerfile .
                '''
            }
        }

        stage('Push Docker Image') {
            when { 
                expression { 
                    return env.GIT_BRANCH == 'origin/main'
                }
            }
            steps {
                sh '''
                docker push rosinebelle/cicdproject:${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
        success {
            slackSend (
                channel: '#development-alerts',
                color: 'good',
                message: "SUCCESSFUL: Application s7rosine-do-it-yourself-cart Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }

        unstable {
            slackSend (
                channel: '#development-alerts',
                color: 'warning',
                message: "UNSTABLE: Application s7rosine-do-it-yourself-cart Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }

        failure {
            slackSend (
                channel: '#development-alerts',
                color: '#FF0000',
                message: "FAILURE: Application s7rosine-do-it-yourself-cart Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }

        cleanup {
            deleteDir()
        }
    }
}
