pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('DockerHub-Cred')
        SONARQUBE_ENV = 'Sonar'
    }

    tools {
        maven 'maven' // Ensure Maven is properly defined in Jenkins tools
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/s7rosine/ci-cd-project.git'
            }
        }

        stage('Test') {
            steps {
                echo 'Running Maven tests...'
                sh 'mvn clean test'
                sh 'mvn package'
                sh 'ls -la target' // Check if the target directory exists
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
                docker build -t rosinebelle/cicdproject:v1.0.0 .
                '''
            }
        }

        stage('Push Docker Image') {
            when { 
                expression { 
                    return env.BRANCH_NAME == 'main' // Ensure BRANCH_NAME is correctly used
                }
            }
            steps {
                sh '''
                docker push rosinebelle/cicdproject:v1.0.0
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker run -itd --name ci-cd-project rosinebelle/cicdproject:v1.0.0
                docker ps | grep ci-cd-project
                '''
            }
        }
    }

    post {
        success {
            slackSend (
                channel: '#development-alerts',
                color: 'good',
                message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }

        unstable {
            slackSend (
                channel: '#development-alerts',
                color: 'warning',
                message: "UNSTABLE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }

        failure {
            slackSend (
                channel: '#development-alerts',
                color: '#FF0000',
                message: "FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }

        cleanup {
            deleteDir()
        }
    }
}
