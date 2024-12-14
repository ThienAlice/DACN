
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

        DOMAIN = "http://lixsong.click/"

        NEXUS_URL = "http://nexus.thienngo.tech"
        ARTIFACT_NAME = "${PROJECT_NAME}-build-${TAG_IMAGE}.tar.gz"

        HARBOR_REPO_NAME="devsecops"
        TAG_IMAGE= "${env.BUILD_ID}"
        HARBOR_IMAGE="${HARBOR_REPO_NAME}/${PROJECT_NAME}:${TAG_IMAGE}"

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
            when{
                branch 'develop'
            }
            steps{
                echo "Unit test Success"
            }
        }
        stage('SonarQube Analysis') {
            when{
                branch 'develop'
            }
            steps {
                dir("${FOLDER_PATH}") {
                    withSonarQubeEnv('sonar') {  h
                        sh "${SCANNER_HOME}/bin/sonar-scanner"
                    }
                }
            }
        }
        stage('Quality Gate') {
            when{
                branch 'develop'
            }
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
            when{
                branch 'develop'
            }
            steps {
                sh """trivy fs --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o ${TRIVY_FS_REPORT} ${FOLDER_PATH} """
            }
        }
        stage('Snyk Security Test') {
            when{
                branch 'develop'
            }
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
            when{
                branch 'develop'
            }
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
        stage('Push Docker Image') {
            steps {
                withDockerRegistry(credentialsId: 'harbor-cred', url: 'https://harbor.thienngo.click/') {
                    sh """docker tag ${IMAGE_VERSION} harbor.thienngo.click/devsecops/${IMAGE_VERSION}"""
                    sh """docker push harbor.thienngo.click/devsecops/${IMAGE_VERSION}"""
                }
            }
        }
        stage('Clone Config Repo') {
            steps {
                git branch: 'main',
                credentialsId: 'github-token-config',
                url: 'https://github.com/baolonggg/Config-DACN.git'
            }
        }
        stage('Update Manifest') {
            environment {
                GIT_REPO_NAME = "Config-DACN"
                GIT_USER_NAME = "baolonggg"
            }
            steps {
                dir('Config/infra1') {
                    withCredentials([string(credentialsId: 'github-token-config', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                            git config user.email "thienngo.081003@gmail.com"
                            git config user.name "ThienAlice"
                            echo $TAG_IMAGE
                            imageTag=$(grep -oP '(?<=devsecops/microservice/order-service:)[^ ]+' order-service.deployment.yaml)
                            echo $imageTag
                            sed -i "s#${HARBOR_IMAGE}:${imageTag}#${HARBOR_IMAGE}:$TAG_IMAGE#" order-service.deployment.yaml
                            git add order-service.deployment.yaml
                            git commit -m "Update deployment Image to version \${TAG_IMAGE}"
                            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        '''
                    }
                }
            }
        }
        stage('DAST Scan') {
            steps {
                sh 'mkdir dast-report'
                sh """docker run --rm -v ${env.WORKSPACE}/dast-report:/tmp/ thien0810/arachni:v1.4-0.5.10 bin/arachni --output-verbose --scope-include-subdomains ${DOMAIN} --report-save-path=/tmp/microservice.afr """
                sh """docker run --rm -v ${env.WORKSPACE}/dast-report:/tmp/ thien0810/arachni:v1.4-0.5.10 bin/arachni_reporter /tmp/microservice.afr --reporter=html:outfile=/tmp/microservice.html.zip"""
                sh 'sudo chown -R jenkins:jenkins dast-report && sudo chmod 777 dast-report'
                sh 'cd dast-report && unzip microservice.html.zip && rm -rf *.zip *.afr '
            }
        }

    }
    post {
        always {
            archiveArtifacts artifacts: "${TRIVY_FS_REPORT}", allowEmptyArchive: true
            archiveArtifacts artifacts: "${TRIVY_IMAGE_REPORT}", allowEmptyArchive: true
            archiveArtifacts artifacts: "dast-report/**/*", allowEmptyArchive: true
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

