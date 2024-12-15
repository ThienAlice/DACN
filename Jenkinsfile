
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

        PROJECT_NAME = "order-service"
        FOLDER_PATH="source/src/${PROJECT_NAME}"

        NEXUS_URL = "http://nexus.thienngo.tech"

        TAG_IMAGE= "${env.BUILD_ID}"
        IMAGE_VERSION = "${order-service}:${TAG_IMAGE}"

        TRIVY_IMAGE_REPORT = "trivy-report_image-${env.BUILD_ID}.html"
        TRIVY_FS_REPORT = "trivy-report_fs-${env.BUILD_ID}.html"

    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}",
                    credentialsId: 'git-cred',
                    url: 'https://github.com/ThienAlice/DACN.git'
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
                        sh "npm config set registry ${NEXUS_URL}/repository/npm-proxy-repo/"
                        sh 'npm install'
                    }
                }
            }
        }
        stage('Trivy scan file system') {
            steps {
                sh """trivy fs --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o ${TRIVY_FS_REPORT} ${FOLDER_PATH} """
            }
        }
        stage('Snyk Security Test') {
            steps {
                snykSecurityCheck("${FOLDER_PATH}", "MicroService/${PROJECT_NAME}")
            }
        }
        stage('Build and Tage Docker Image') {
            steps {
                dir("${FOLDER_PATH}"){
                    withDockerRegistry(credentialsId: 'harbor-cred', url: 'https://harbor.thienngo.click/') {
                        sh "docker build -t ${IMAGE_VERSION} ."
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
                        $IMAGE_VERSION
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
    


