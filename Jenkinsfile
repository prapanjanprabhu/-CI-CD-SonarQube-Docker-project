pipeline {
  agent any

  tools {
    maven 'Maven3.9.12'
  }

  environment {
    DOCKER_IMAGE = "prapanjanprabhu/devops-app"
    SONAR_ENV = "sonarqube"
  }

  stages {

    stage('Build & Test') {
      steps {
        bat 'mvn -B clean test package'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv("${SONAR_ENV}") {
          withCredentials([string(credentialsId: 'sonar-token00', variable: 'SONAR_TOKEN')]) {
            bat '''
              mvn -B org.sonarsource.scanner.maven:sonar-maven-plugin:4.0.0.4121:sonar ^
                -Dsonar.login=%SONAR_TOKEN%
            '''
          }
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 10, unit: 'MINUTES') {
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
