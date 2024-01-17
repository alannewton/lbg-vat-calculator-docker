pipeline {

    environment {
        registry = "alannewton/vatcal"
        registryCredentials = "dockerhub_id"
        dockerImage = ""
    }

    agent any

    stages {

        stage ('Build Docker Image') {
            steps{
                script {
                    dockerImage = docker.build(registry)
                }
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/alannewton/lbg-vat-calculator-docker.git'
            }
        }

        stage('Install') {
            steps {
                sh "npm install"
            }
        }

        stage('Test') {
            steps {
                sh "npm test"
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'sonarqube'
            }

            steps {
                withSonarQubeEnv('sonarqube-alan') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage ("Push to Docker Hub"){
            steps {
                script {
                    docker.withRegistry('', registryCredentials) {
                        dockerImage.push("${env.BUILD_NUMBER}")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage ("Clean up"){
            steps {
                script {
                    sh 'docker image prune --all --force --filter "until=48h"'
                        }
            }
        }
    }
}