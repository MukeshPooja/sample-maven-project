pipeline {
    agent any
    
    tools {
        jdk 'JDK-17'
        maven 'Maven-3.9'
    }
    
    environment {
        SONAR_HOST_URL = 'http://54.242.3.165:9000'
        SONAR_PROJECT_KEY = 'sample-maven-project'
        // If using Jenkins credentials:
        // SONAR_TOKEN = credentials('sonarqube-token')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "✅ Checked out: ${env.GIT_COMMIT}"
            }
        }
        
        stage('Build & Test') {
            steps {
                echo '🔨 Building with Maven...'
                sh 'mvn clean verify'
            }
            post {
                success {
                    echo '✅ Build & tests passed'
                    archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: false
                }
                failure {
                    error '❌ Build or tests failed'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo '🔍 Running SonarQube analysis...'
                withSonarQubeEnv('SonarQube-Prod') {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.login=${SONAR_AUTH_TOKEN}
                    '''
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                echo '🚦 Waiting for SonarQube Quality Gate...'
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
            post {
                success {
                    echo '✅ Quality Gate passed - code meets standards'
                }
                failure {
                    echo '❌ Quality Gate failed - fix issues and re-run'
                    // Optional: Send notification
                }
            }
        }
        
        stage('Package for Deployment') {
            when {
                branch 'main'
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                echo '📦 Packaging application...'
                sh 'mvn package -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: false
                    echo '✅ Artifact ready for deployment'
                }
            }
        }
    }
    
    post {
        always {
            echo '🧹 Cleaning workspace...'
            cleanWs()
        }
        success {
            echo '🎉 Pipeline completed successfully!'
        }
        failure {
            echo '💥 Pipeline failed - check console output'
        }
    }
}