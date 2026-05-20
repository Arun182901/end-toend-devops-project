pipeline {
    agent any
    environment {
        IMAGE_NAME = "arunsin/flask-devops"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        // stage('OWASP Dependency Check') {
        //     steps {
        //         dependencyCheck odcInstallation: 'Dependency-Check', additionalArguments: '--scan ./'
        //     }
        // }
    stage('SonarQube Analysis') {
        steps {
            script {
            def scannerHome = tool 'sonar-scanner'

            withSonarQubeEnv('sonarqube') {
               withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
               sh """
               ${scannerHome}/bin/sonar-scanner \
               -Dsonar.projectKey=flask-app-py \
               -Dsonar.sources=. \
               -Dsonar.host.url=http://34.122.36.4:9000 \
               -Dsonar.login=${SONAR_TOKEN}
               """
            }
          }
        }
    }
}
        
        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }
        stage('Trivy Scan') {
            steps {
                sh 'trivy image $IMAGE_NAME:$BUILD_NUMBER'
            }
        }
        stage('Docker Push') {
            steps {
                withDockerRegistry([url: 'https://index.docker.io/v1/', credentialsId: 'dockerhub']) {
                    sh 'docker push $IMAGE_NAME:$BUILD_NUMBER'
                }
            }
        }
    }
}
