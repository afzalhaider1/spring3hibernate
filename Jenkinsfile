pipeline {
    agent any

parameters {
    booleanParam(name: 'SKIP_CODE_STABILITY', defaultValue: false, description: 'Skip Code Stability Analysis')
    booleanParam(name: 'SKIP_SONARQUBE_CODE_QUALITY_ANALYSIS', defaultValue: false, description: 'Skip SonarQube Analysis')
    booleanParam(name: 'SKIP_CODE_COVERAGE', defaultValue: false, description: 'Skip Code Coverage Analysis')
}
    stages {
        stage('Code Checkout') {
            steps {
                git 'https://github.com/afzalhaider1/spring3hibernate.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Parallel Stages') {   
            parallel {
            
            stage('Code Stability') {
                when {
                    expression { params.SKIP_CODE_STABILITY != true }
                }
                steps {
                    sh 'mvn pmd:pmd'
                }
            }
        
            stage('SonarQube code quality analysis') {
                when {
                    expression { params.SKIP_SONARQUBE_CODE_QUALITY_ANALYSIS != true }
                }
                steps {
                    withSonarQubeEnv('sonarQube') {
                        sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                    }
                }
        }
            stage('Code Coverage Analysis') {
            when {
                expression { params.SKIP_CODE_COVERAGE != true }
            }
            steps {
                sh 'mvn jacoco:report'
            }
        }
    }
  }
    stage('Input') {
            steps {
                input('Do you want to proceed?')
            }
        }
        stage('Publish Artifacts, If proceed') {
            steps {
                script {
                    // This stage will be executed only if the user proceeds in the "Input" stage.
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: '172.31.10.132:8081',
                        groupId: 'Spring3HibernateApp',
                        version: '4.0.0',
                        repository: 'spring3hibernate',
                        credentialsId: 'nexus',
                        artifacts: [
                            [artifactId: 'Spring3HibernateApp',
                             classifier: '',
                             file: 'target/Spring3HibernateApp.war',
                             type: 'war']
                        ]
                    )
                }
            }
        }
    }
    post {
        failure {
            // Send email notification on job failure
            emailext body: "The build failed. Check the console output: ${BUILD_URL}",
                    subject: "[FAILURE] Build failed: ${currentBuild.fullDisplayName}",
                    to: "maiafzal@gmail.com"
            // Send Slack notification on job failure
            slackSend(color: "danger", message: "The build failed. Check the console output: ${BUILD_URL}")
        }
        success {
            // Send email notification on job success
            emailext body: "The build succeeded. View the artifacts: ${BUILD_URL}",
                    subject: "[SUCCESS] Build succeeded: ${currentBuild.fullDisplayName}",
                    to: "maiafzal@gmail.com"
            // Send Slack notification on job success
            slackSend(color: "good", message: "The build succeeded. View the artifacts: ${BUILD_URL}")
        }
    }
}
