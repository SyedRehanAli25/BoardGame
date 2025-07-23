pipeline {
    agent any

    tools {
        maven 'LocalMaven'
        jdk 'JDK'
    }

    environment {
        SONARQUBE_SERVER = 'MySonarQubeServer'  // Match Jenkins global config name
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/adarshadshetty/BoardGame.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }
    }
}

