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
                          -Dsonar.login=${SONAR_AUTH_TOKEN} \
                          -Dsonar.sources=src/main/java \
                          -Dsonar.tests=src/test/java \
                          -Dsonar.java.binaries=target/classes \
                          -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                    '''
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                echo '🚦 Waiting for SonarQube Quality Gate...'
                script {
                    // Wait for quality gate with proper timeout
                    timeout(time: 15, unit: 'MINUTES') {
                        def qualityGateStatus = waitForQualityGate()
                        echo "Quality Gate Result: ${qualityGateStatus}"
                    }
                }
            }
            post {
                success {
                    echo '✅ Quality Gate passed - code meets standards'
                }
                failure {
                    echo '❌ Quality Gate failed - fix issues and re-run'
                }
            }
        }
        // ✅ stages block closes HERE
    }
    
    // ✅ Pipeline-level post block (outside stages)
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