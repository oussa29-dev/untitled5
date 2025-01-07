pipeline {
    agent any

    tools {
        jdk 'JDK11'  // Assurez-vous que le JDK est configuré
    }

    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_TOKEN = 'your-sonar-token'
        MAVEN_REPO_URL = 'https://mymavenrepo.com/'
        MAVEN_REPO_USER = 'username'
        MAVEN_REPO_PASSWORD = 'password'
        SLACK_CHANNEL = '#your-slack-channel'
    }

    stages {
        stage('Test') {
            steps {
                script {
                    echo 'Running Unit Tests...'
                    sh './gradlew test'
                }
                archiveArtifacts artifacts: '**/build/reports/tests/*.html', allowEmptyArchive: true
                cucumber buildStatus: 'SUCCESS', fileIncludePattern: '**/build/reports/cucumber/*.json'
            }
        }
        stage('Code Analysis') {
            steps {
                script {
                    echo 'Running SonarQube Analysis...'
                    sh './gradlew sonarqube'
                }
            }
        }
        stage('Code Quality') {
            steps {
                script {
                    echo 'Checking Quality Gates...'
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Pipeline stopped due to Quality Gate failure: ${qg.status}"
                    }
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    echo 'Building JAR and Documentation...'
                    sh './gradlew jar'
                    sh './gradlew javadoc'
                }
                archiveArtifacts artifacts: '**/build/libs/*.jar', allowEmptyArchive: false
                archiveArtifacts artifacts: '**/build/docs/javadoc/**/*', allowEmptyArchive: false
            }
        }
        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying to Maven Repository...'
                    sh """
                    curl -u ${MAVEN_REPO_USER}:${MAVEN_REPO_PASSWORD} \
                        -T build/libs/*.jar ${MAVEN_REPO_URL}
                    """
                }
            }
        }
        stage('Notification') {
            steps {
                script {
                    echo 'Sending Notifications...'
                    mail to: 'team@example.com',
                         subject: 'Deployment Successful',
                         body: 'The application has been successfully deployed.'
                    slackSend channel: "${SLACK_CHANNEL}", message: 'Deployment successful!'
                }
            }
        }
    }

    post {
        failure {
            script {
                echo 'Pipeline Failed. Sending Failure Notification...'
                mail to: 'team@example.com',
                     subject: 'Pipeline Failed',
                     body: 'The pipeline failed. Please check Jenkins logs.'
                slackSend channel: "${SLACK_CHANNEL}", message: 'Pipeline failed!'
            }
        }
    }
}
