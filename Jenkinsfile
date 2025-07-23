pipeline {
    agent any

    parameters {
        booleanParam(
            name: 'RUN_SONAR_SCAN',
            defaultValue: true,
            description: 'Run SonarQube Code Analysis'
        )
    }

    tools {
        jdk 'JDK17'           // Must match Jenkins config
        maven 'LocalMaven'    // Must match Maven installation name in Jenkins
    }

    environment {
        SONAR_URL = 'http://localhost:9000'     // Replace if you're using another host
        SONAR_AUTH_TOKEN = credentials('Jenkins_token') // Your SonarQube token ID
        EMAIL_RECIPIENT = 'rehan.ali9325@gmail.com'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/SyedRehanAli25/BoardGame.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Parallel Quality Checks') {
            parallel {
                stage('Code Stability') {
                    steps {
                        sh 'mvn verify'
                    }
                }
                stage('Code Quality (SonarQube)') {
                    when {
                        expression { return params.RUN_SONAR_SCAN }
                    }
                    steps {
                        withSonarQubeEnv('MySonarQubeServer') {
                            sh "mvn sonar:sonar -Dsonar.projectKey=BoardGame -Dsonar.login=$SONAR_AUTH_TOKEN"
                        }
                    }
                }
                stage('Code Coverage') {
                    steps {
                        sh 'mvn test'
                    }
                }
            }
        }

        stage('Publish Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }

    post {
        success {
            mail to: "${EMAIL_RECIPIENT}",
                 subject: "Jenkins Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Success! Jenkins build completed.\n\nProject: ${env.JOB_NAME}\nBuild: ${env.BUILD_NUMBER}\nURL: ${env.BUILD_URL}"
        }
        failure {
            mail to: "${EMAIL_RECIPIENT}",
                 subject: "Jenkins Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Failed! Jenkins build encountered an error.\n\nProject: ${env.JOB_NAME}\nBuild: ${env.BUILD_NUMBER}\nURL: ${env.BUILD_URL}"
        }
    }
}

