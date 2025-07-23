pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven'
    }

    stages {
        stage('Parallel Code Checks') {
            parallel {
                stage('Code Stability') {
                    steps {
                        echo 'Running unit tests for code stability...'
                        sh 'mvn test'
                    }
                }
                stage('Code Quality Analysis') {
                    steps {
                        echo 'Running SonarQube analysis...'
                        withSonarQubeEnv('MySonarQube') {
                            sh 'mvn sonar:sonar'
                        }
                    }
                }
                stage('Code Coverage Analysis') {
                    steps {
                        echo 'Analyzing code coverage...'
                        sh 'mvn jacoco:report'
                    }
                }
            }
        }

        stage('Generate Reports') {
            steps {
                echo 'Generating aggregated reports...'
                // Optional: Combine reports or post-process
            }
        }

        stage('Publish Artifacts') {
            steps {
                echo 'Publishing build artifacts...'
                sh 'mvn package'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }
}

