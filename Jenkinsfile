
@Library('jenkins-libraries') _
pipeline {
    agent { label 'build-server' }
    parameters {
        choice(name: 'SERVICE', choices: ['order-service', 'product-service', 'store-front'], description: 'Choose service'),
    }
    tools {
        jdk 'jdk17'
        nodejs 'node18'
    }
    environment {
        VERSION  = "0.0.1"

        SCANNER_HOME = tool 'sonar-scanner'

        PROJECT_NAME = "${params.SERVICE}"
        FOLDER_PATH="source/src/${PROJECT_NAME}"

        TAG_IMAGE= "${env.BUILD_ID}"
        

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
                    withSonarQubeEnv('sonar') {  h
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
                        if (params.SERVICE == 'order-service' || params.SERVICE == 'store-front' ){
                            sh "npm config set registry ${NEXUS_URL}/repository/npm-proxy-repo/"
                            sh "npm install"
                            if (params.SERVICE == 'store-front'){
                                sh "npm run build"
                            }
                        }
                        else if (params.SERVICE == 'product-service'){
                            sh "cargo build"
                        }
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
                snykSecurityCheck("${FOLDER_PATH}", 'MicroService/${params.SERVICE}')
            }
        }
        stage('Push artifact to Nexus') {
            steps {
                script{
                    dir("$FOLDER_PATH"){
                        if (params.SERVICE == 'product-service' || params.SERVICE == 'store-front'){
                            sh 'tar -czvf ${ARTIFACT_NAME} build'
                            sh "curl -u ${NEXUS_USERNAME}:${NEXUS_PASSWORD} --upload-file ${ARTIFACT_NAME} ${NEXUS_URL}/repository/microservice-artifact-repo/${PROJECT_NAME}/build/${VERSION}/${ARTIFACT_NAME}"
                        }
                    }
                }
            }
        }
        stage('Build and Tage Docker Image') {
            steps {
                dir("${FOLDER_PATH}"){
                    withDockerRegistry(credentialsId: 'harbor-cred', url: 'https://harbor.thienngo.click/') {
                        sh """docker build -t ${IMAGE_VERSION} ."""
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
            echo "Pipeline for ${params.SERVICE} completed successfully!"
        }
        failure {
            echo "Pipeline for ${params.SERVICE} failed."
        }
    }
}

