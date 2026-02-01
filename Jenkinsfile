pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "dockerhubusername/devops-app"
        SONAR_ENV = "sonarqube"
    }

    stages {



        stage('Build & Test') {
            steps {
                bat 'mvn clean test package'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv("${SONAR_ENV}") {
                    bat """
                    mvn sonar:sonar ^
                    -Dsonar.projectKey=devops-app ^
                    -Dsonar.host.url=http://localhost:9000
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:latest .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    docker push %DOCKER_IMAGE%:latest
                    """
                }
            }
        }

        stage('Deploy Container') {
            steps {
                bat """
                docker stop devops-app || exit 0
                docker rm devops-app || exit 0
                docker run -d --name devops-app -p 8081:8080 %DOCKER_IMAGE%:latest
                """
            }
        }
    }
}




