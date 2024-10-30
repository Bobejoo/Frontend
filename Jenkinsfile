def imageName="192.168.44.44:8082/docker_registry/frontend"
def dockerRegistry="https://192.168.44.44:8082"
def registryCredentials="artifactory"

pipeline {
    agent {
        label 'agent'
    }
    environment {
        PIP_BREAK_SYSTEM_PACKAGES = 1
        scannerHome = tool 'SonarQube'
    }
    stages {
        stage('Get Frontend') {
            steps {
                checkout scm
            }
        }
    
        stage('Install requirements and run tests') {
            steps {
                sh """pip3 install -r requirements.txt
                      python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml"""
            }
        }
        stage('Sonar snooping') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage('Build app image') {
            steps {
                script {
                  applicationImage = docker.build("${imageName}:${env.BUUILD_ID}")
                }
            }
        }
        stage ('Push image to repo') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }
            }
        }
    }
    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }
}
