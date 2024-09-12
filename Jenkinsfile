pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('DockerHub-Cred')
        SONARQUBE_ENV = 'Sonar'
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
                mvn clean test
                mvn package
                
                '''
            }
        }

        stage('SonarQube Analysis') {
            agent {
                docker { image 'maven:3.8.5-openjdk-18' }
            }
            steps {
                script {
                    echo 'Running SonarQube analysis...'
                    withSonarQubeEnv("${env.SONARQUBE_ENV}") {
                        sh 'mvn sonar:sonar'
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
                docker build -t rosinebelle/cicdproject:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Push Docker Image') {
            when { 
                expression { 
                    return env.BRANCH_NAME == 'main'
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
                message: "SUCCESSFUL: Application s7rosine-ci-cd-project Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }

        unstable {
            slackSend (
                channel: '#development-alerts',
                color: 'warning',
                message: "UNSTABLE: Application s7rosine-ci-cd-project Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }

        failure {
            slackSend (
                channel: '#development-alerts',
                color: '#FF0000',
                message: "FAILURE: Application s7rosine-ci-cd-project Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }

        cleanup {
            deleteDir()
        }
    }
}
