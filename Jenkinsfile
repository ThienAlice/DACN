
@Library('jenkins-libraries') _
pipeline {
    agent { label 'build-server' }
    tools {
        jdk 'jdk17'
        nodejs 'node18'
    }
    environment {
        VERSION  = "0.0.1"

        SCANNER_HOME = tool 'sonar-scanner'
        NEXUS_URL = "http://nexus.thienngo.tech"
        TAG_IMAGE= "${env.BUILD_ID}"
        
        TRIVY_IMAGE_REPORT = "trivy-report_image-${env.BUILD_ID}.html"
        TRIVY_FS_REPORT = "trivy-report_fs-${env.BUILD_ID}.html"

    }
    stages {
        stage ('Prepare') {
            steps {
                script {
                    env.PROJECT_NAME = myLibrary.getProjectName("${env.BRANCH_NAME}")
                    env.FOLDER_PATH="source/src/${PROJECT_NAME}"
                    env.IMAGE_DOCKER = "${PROJECT_NAME}:${TAG_IMAGE}"
                }
            }
        }
        stage ('Unit Test'){
            steps{
                echo "Unit test Success"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                dir("${FOLDER_PATH}") {
                    withSonarQubeEnv('sonar') {  
                        sh "${SCANNER_HOME}/bin/sonar-scanner"
                    }
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false,
                        credentialsId: 'sonar-token'
                }
            }
        }
        stage('Install Dependencies And Build') {
            steps {
                script {
                    dir("${FOLDER_PATH}") {     
                        sh "cargo build"
                    }
                }
            }
        }
        stage('Trivy scan file system') {
            steps {
                sh """trivy fs --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o ${TRIVY_FS_REPORT} ${FOLDER_PATH} """
            }
        }
        stage('Scan Depedencies') {
            steps {
                dir("${FOLDER_PATH}") { 
                    script {                    
                        def auditStatus = sh(script: 'cargo audit --json > audit-report.json', returnStatus: true)
                        if (auditStatus == 0) {
                            echo "No vulnerabilities found."
                        }else {
                            echo "Vulnerabilities detected! Review the audit-report.json."
                        }
                    }
                    archiveArtifacts artifacts: 'audit-report.json', allowEmptyArchive: true
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                dir("${FOLDER_PATH}"){
                    withDockerRegistry(credentialsId: 'harbor-cred', url: 'https://harbor.thienngo.click/') {
                        sh """docker build -t ${IMAGE_DOCKER}  ."""
                    }
                }
            }
        }
        stage('Docker Image Scan') {
            steps {
                sh "trivy clean --all"
                sh """
                    trivy image --timeout 10m \
                    --format template \
                    --template "@/usr/local/share/trivy/templates/html.tpl" \
                    --output ./${TRIVY_IMAGE_REPORT} \
                        ${IMAGE_DOCKER}
                    """
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: "${TRIVY_FS_REPORT}", allowEmptyArchive: true
            archiveArtifacts artifacts: "${TRIVY_IMAGE_REPORT}", allowEmptyArchive: true
            cleanWs()
        }
        success {
            echo "Pipeline for ${PROJECT_NAME} completed successfully!"
        }
        failure {
            echo "Pipeline for ${PROJECT_NAME} failed."
        }
    }
}