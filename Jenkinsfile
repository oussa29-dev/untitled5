pipeline {
    agent any // This specifies that the pipeline can run on any available agent

    environment {
        SONARQUBE = 'sonar'  // This must match the name you gave in Jenkins config
        username = "myMavenRepo"  // Replace with your actual Jenkins credential ID
        password= "test0005" // Replace with your actual Jenkins credential ID
    }
   
    stages {
        // Stage for running tests
        stage('TEST') {
            steps {
                script {
                    echo 'Running tests...'
                    // Run your tests here, e.g. using Gradle or Maven
                    sh 'chmod +x gradlew'
                    sh './gradlew test'


                }
            }
        }


      
        // Stage for Code Quality (SonarQube Quality Gate Check)
        stage('CODEQUALITY') {
            steps {
                script {
                    echo 'Checking SonarQube Quality Gate...'
                    def qualityGate = waitForQualityGate()
                    if (qualityGate.status != 'OK') {
                        error "SonarQube Quality Gate failed: ${qualityGate.status}"
                    }
                }
            }
        }

        // Stage for building the project
        stage('BUILD') {
            steps {
                script {
                    echo 'Building the project...'
                    // Build your project, e.g., using Gradle or Maven
                    sh './gradlew build'
                }
            }
        }

        // Stage for deploying the application
        stage('DEPLOY') {
            steps {
                script {
                    echo 'Deploying the application...'
                    sh './gradlew publish'  // Customize with your deployment command
                }
            }
        }

        // Stage for notifications (e.g., email, Slack)
        stage('NOTIFY') {
            steps {
                script {
                    echo 'Sending email notification...'
                    // Call Gradle sendMail task
                    sh './gradlew sendMail'
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
