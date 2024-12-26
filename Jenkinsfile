
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

        HARBOR_REPO_NAME="devsecops"
        
        PATH = "/home/jenkins/.cargo/bin:$PATH"

        DOMAIN = "http://lixsong.click/"

    }
    stages {
        stage('Detect Changes') {
            steps {
                script {
                    // Lấy danh sách các thư mục thay đổi
                    def changedDirs = sh(script: """git diff --name-only HEAD~1 HEAD | awk -F/ '{print \$3}' | sort | uniq""", returnStdout: true).trim().split('\n')
                    echo "Changed Directories: ${changedDirs}"
                    env.CHANGED_DIRS = changedDirs.join(',')
                }
            }
        }
        stage('Run Pipelines for Changed Services') {
            steps {
                script {
                    def dirs = env.CHANGED_DIRS.split(',')
                    // dirs.each { dir ->
                    //     echo "Running pipeline for: ${dir}"
                    //     // Triển khai logic cho từng service ở đây
                    //     if (dir == 'store-front') {
                    //         build job: 'MicroserviceWeb/job/store-front-service', parameters: [string(name: 'GIT_COMMIT', value: env.GIT_COMMIT)]
                    //     } else if (dir == 'order-service') {
                    //         build job: 'MicroserviceWeb/job/order-service', parameters: [string(name: 'GIT_COMMIT', value: env.GIT_COMMIT)]
                    //     }
                    // }
                    build job: "../store-front-service", parameters: 
                    [
                        string(name: 'GIT_COMMIT', value: env.GIT_COMMIT),
                        string(name: 'BRANCH_NAME', value: env.BRANCH_NAME),
                    ]
                }
            }
        }       
        // stage ('Prepare') {
        //     steps {
        //         script {
        //             env.PROJECT_NAME = myLibrary.getProjectName("${env.BRANCH_NAME}")
        //             env.FOLDER_PATH="source/src/${PROJECT_NAME}"
        //             env.IMAGE_DOCKER = "microservice/${PROJECT_NAME}:${TAG_IMAGE}"
        //             env.ARTIFACT_NAME = "${PROJECT_NAME}-build-${TAG_IMAGE}.tar.gz"
        //         }
        //     }
        // }
        // stage ('Unit Test'){
        //     when {
        //         expression { env.BRANCH_NAME != 'main' }
        //     }
        //     steps{
        //         echo "Unit test Success"
        //     }
        // }
        // stage ('Integration Test'){
        //     when {
        //         expression { env.BRANCH_NAME == 'test' }
        //     }
        //     steps{
        //         echo "Integration Test Success"
        //     }
        // }
        // stage('SonarQube Analysis') {
        //     when {
        //         expression { env.BRANCH_NAME.startsWith('feature')}
        //     }
        //     steps {
        //         dir("${FOLDER_PATH}") {
        //             withSonarQubeEnv('sonar') {  
        //                 sh "${SCANNER_HOME}/bin/sonar-scanner"
        //             }
        //         }
        //     }
        // }
        // stage('Quality Gate') {
        //     when {
        //         expression { env.BRANCH_NAME.startsWith('feature')}
        //     }
        //     steps {
        //         script {
        //             waitForQualityGate abortPipeline: false,
        //                 credentialsId: 'sonar-token'
        //         }
        //     }
        // }
        // stage('Install Dependencies And Build') {
        //     steps {
        //         script {
        //             if (env.BRANCH_NAME.startsWith('feature'))
        //             {
        //                 def typeProject = myLibrary.typeService("${env.BRANCH_NAME}")
        //                 dir("${FOLDER_PATH}") {     
        //                     if (typeProject == "js") {
        //                         sh "npm config set registry ${NEXUS_URL}/repository/npm-proxy-repo/"
        //                         sh "npm install"
        //                         if (env.BRANCH_NAME == 'feature/store-front') {
        //                             sh "npm run build"
        //                         }
        //                     } 
        //                     else if (typeProject == "go") {
        //                         sh "cargo build"
        //                     }
        //                 }
        //             }
        //             else {
                        
        //             }
        //         }
        //     }
        // }
        // stage('Push artifact to Nexus') {
        //     when {
        //         expression { env.BRANCH_NAME == "main" && env.PROJECT_NAME != 'order-service'}
        //     }
        //     steps {
        //         dir("${FOLDER_PATH}"){
        //             script {
        //                 if (env.PROJECT_NAME == 'store-front') {
        //                     sh 'tar -czvf ${ARTIFACT_NAME} dist'
        //                 }
        //                 else {
        //                    sh 'tar -czvf ${ARTIFACT_NAME} -C target/debug product-service'
        //                 }
        //                 sh """curl -u ${NEXUS_USERNAME}:${NEXUS_PASSWORD} --upload-file ${ARTIFACT_NAME} ${NEXUS_URL}/repository/microservice-artifact-repo/${PROJECT_NAME}/build/${VERSION}/${ARTIFACT_NAME}"""
        //             }
        //         }
                
        //     }
        // }
        // stage('Trivy scan file system') {
        //     when {
        //         expression { env.BRANCH_NAME.startsWith('feature')}
        //     }
        //     steps {
        //         sh """trivy fs --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o ${TRIVY_FS_REPORT} ${FOLDER_PATH} """
        //     }
        // }
        // stage('Scan Depedencies') {
        //     when {
        //         expression { env.BRANCH_NAME.startsWith('feature')}
        //     }
        //     steps {
        //         dir("${FOLDER_PATH}") { 
        //             script {
        //                 if(env.PROJECT_NAME == 'store-front' || env.PROJECT_NAME == 'order-service') {
        //                     myLibrary.snykSecurityCheck("${FOLDER_PATH}", "${PROJECT_NAME}")
        //                 } else {
        //                     def auditStatus = sh(script: 'cargo audit --json > audit-report.json', returnStatus: true)
        //                     if (auditStatus == 0) {
        //                         echo "No vulnerabilities found."
        //                     }else {
        //                         echo "Vulnerabilities detected! Review the audit-report.json."
        //                     }
        //                     archiveArtifacts artifacts: 'audit-report.json', allowEmptyArchive: true
        //                 }
        //             }
                    
        //         }
        //     }
        // }
        // stage('Build Docker Image') {
        //     steps {
        //         dir("${FOLDER_PATH}"){
        //             withDockerRegistry(credentialsId: 'harbor-cred', url: 'https://harbor.thienngo.click/') {
        //                 sh "docker build -t ${IMAGE_DOCKER}  ."
        //             }
        //         }
        //     }
        // }
        // stage('Docker Image Scan') {
        //     when {
        //         expression { env.BRANCH_NAME.startsWith('feature')}
        //     }
        //     steps {
        //         sh "trivy clean --all"
        //         sh """
        //             trivy image --timeout 10m \
        //             --format template \
        //             --template "@/usr/local/share/trivy/templates/html.tpl" \
        //             --output ./${TRIVY_IMAGE_REPORT} \
        //                 ${IMAGE_DOCKER}
        //             """
        //     }
        // }
        // stage('Push Docker Image') {
        //     when {
        //         expression { env.BRANCH_NAME == 'test'||env.BRANCH_NAME == 'main'}
        //     }
        //     steps {
        //         script {
        //             withDockerRegistry(credentialsId: 'harbor-cred', url: 'https://harbor.thienngo.click/') { 
        //                 if (env.BRANCH_NAME == 'test') {
        //                     HARBOR_REPO_NAME="devsecops-test"
        //                 }
        //                 sh "docker tag ${IMAGE_DOCKER} harbor.thienngo.click/${HARBOR_REPO_NAME}/${IMAGE_DOCKER}"
        //                 sh "docker push harbor.thienngo.click/${HARBOR_REPO_NAME}/${IMAGE_DOCKER}"
        //             }
        //         }
        //     }
        // }
        // stage('Update Manifest') {
        //     when {
        //         expression { env.BRANCH_NAME == 'test'||env.BRANCH_NAME == 'main'}
        //     }
        //     environment {
        //         GIT_REPO_NAME = "Config-DACN"
        //         GIT_USER_NAME = "ThienAlice"
        //     }
        //     steps {
        //         git branch: "${env.BRANCH_NAME}", 
        //         credentialsId: 'github-token-config', 
        //         url: 'https://github.com/baolonggg/Config-DACN.git'

        //         dir('Config/infra1') {
        //             withCredentials([string(credentialsId: 'github-token-config', variable: 'GITHUB_TOKEN')]) {
        //                 sh '''
        //                     git config user.email "thienngo.081003@gmail.com"
        //                     git config user.name "ThienAlice"
        //                     echo $TAG_IMAGE
        //                     imageTag=$(grep -oP '(?<=devsecops/microservice/store-front:)[^ ]+' store-front.deployment.yaml)
        //                     echo $imageTag
        //                     sed -i "s#${HARBOR_IMAGE}:${imageTag}#${HARBOR_IMAGE}:$TAG_IMAGE#" store-front.deployment.yaml
        //                     git add store-front.deployment.yaml
        //                     git commit -m "Update deployment Image to version \${TAG_IMAGE}"
        //                     git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
        //                 '''
        //             }
        //         }
        //     }
        // }
        // stage('DAST Scan') {
        //     when {
        //         expression { env.BRANCH_NAME == 'test' || env.BRANCH_NAME == 'main'}
        //     }
        //     steps {
        //         sh 'mkdir dast-report'
        //         sh """docker run --rm -v ${env.WORKSPACE}/dast-report:/tmp/ thien0810/arachni:v1.4-0.5.10 bin/arachni --output-verbose --scope-include-subdomains ${DOMAIN} --report-save-path=/tmp/microservice.afr """
        //         sh """docker run --rm -v ${env.WORKSPACE}/dast-report:/tmp/ thien0810/arachni:v1.4-0.5.10 bin/arachni_reporter /tmp/microservice.afr --reporter=html:outfile=/tmp/microservice.html.zip"""
        //         sh 'sudo chown -R jenkins:jenkins dast-report && sudo chmod 777 dast-report'
        //         sh 'cd dast-report && unzip microservice.html.zip && rm -rf *.zip *.afr '
        //         archiveArtifacts artifacts: "dast-report/**/*", allowEmptyArchive: true
        //     }
        // }
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
