pipeline {
    agent any

    parameters {
        booleanParam(name: 'SKIP_STABILITY', defaultValue: false, description: 'Skip Code Stability Check')
        booleanParam(name: 'SKIP_QUALITY', defaultValue: false, description: 'Skip Code Quality Analysis')
        booleanParam(name: 'SKIP_COVERAGE', defaultValue: false, description: 'Skip Code Coverage Analysis')
    }

    tools {
        maven 'LocalMaven'
        jdk 'JDK'
    }

    environment {
        REPO_URL = 'https://github.com/OT-MICROSERVICES/salary-api.git'
        BRANCH = 'main'
        ARTIFACT_DIR = 'target'
        RECIPIENT = 'rehan.ali9325@gmail.com'
        SLACK_CHANNEL = '#ci-cd-updates'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Parallel Scans') {
            parallel {
                stage('Code Stability') {
                    when {
                        expression { return !params.SKIP_STABILITY }
                    }
                    steps {
                        echo "Running Unit Tests..."
                        sh 'mvn test'
                    }
                }

                stage('Code Quality Analysis') {
                    when {
                        expression { return !params.SKIP_QUALITY }
                    }
                    steps {
                        echo "Running SonarQube Analysis..."
                        withSonarQubeEnv('MySonarQube') {
                            sh 'mvn sonar:sonar'
                        }
                    }
                }

                stage('Code Coverage Analysis') {
                    when {
                        expression { return !params.SKIP_COVERAGE }
                    }
                    steps {
                        echo "Generating Code Coverage Report..."
                        sh 'mvn jacoco:report'
                    }
                }
            }
        }

        stage('Approval to Publish') {
            steps {
                script {
                    def userInput = input message: 'Do you want to publish artifacts?', ok: 'Yes',
                        parameters: [choice(name: 'Proceed?', choices: ['Approve', 'Reject'], description: 'Select an option')]
                    
                    if (userInput == 'Reject') {
                        error "Build Stopped: User rejected artifact publishing."
                    }
                }
            }
        }

        stage('Publish Artifacts') {
            steps {
                echo "Publishing Artifacts..."
                archiveArtifacts artifacts: "${ARTIFACT_DIR}/*.jar", fingerprint: true
            }
        }
    }

    post {
        success {
            mail to: "${RECIPIENT}",
                subject: "Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Good news! The build was successful.\nCheck it here: ${env.BUILD_URL}"

            slackSend channel: "${SLACK_CHANNEL}", message: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }

        failure {
            mail to: "${RECIPIENT}",
                subject: "Build FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Oops! The build failed.\nCheck it here: ${env.BUILD_URL}"

            slackSend channel: "${SLACK_CHANNEL}", message: " FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }

        always {
            echo "Build finished: ${currentBuild.result}"
        }
    }
}
