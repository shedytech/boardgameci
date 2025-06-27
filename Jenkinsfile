pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3.9.9'
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main', 
                    credentialsId: 'github-credentials', 
                    url: 'https://github.com/shedytech/boardgameci.git'
            }
        }

        stage('Test & Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                sh 'mvn deploy'
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-creds', toolName: 'docker') {
                        sh 'docker build -t shedyvince29/boardshack:latest .'
                    }
                }
            }
        }

        stage('Docker Image Scan with Trivy') {
            steps {
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --format table --scanners vuln shedyvince29/boardshack:latest'
            }
        }

        stage('Push Docker Image to DockerHub Registry') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-creds', toolName: 'docker') {
                        sh 'docker push shedyvince29/boardshack:latest'
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
