pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
        
    }
    environment {
        SCANNER_HOME =  tool 'sonar-scanner'
        TRIVY_IMAGE_REPORT = "trivy-fs-report-${env.BUILD_ID}.html"
        CI_PROJECT_NAME = 'boardgame' 
        IMAGE_VERSION = "thienngo/${CI_PROJECT_NAME}:${env.BUILD_ID}"
    }
    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/ThienAlice/DACN.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
               sh "mvn test"
            }
        }
        
        stage('SonarQube Analysic') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=.'''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }
        stage('Snyk Test') {
           steps {
               script {
                   sh "sudo chmod 777 mvnw"
                   snykSecurity failOnError: false, failOnIssues: false, projectName: 'boardgame', snykInstallation: 'snyk', snykTokenId: 'snyk-token', targetFile: 'pom.xml'
                }
            }
        }
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
        stage('Build and Tage Docker Image') {
            steps {
                withDockerRegistry(credentialsId: 'harbor-cred', url: 'https://harbor.thienngo.click/') {
                    sh """docker build -t ${IMAGE_VERSION} ."""
                }
            }
        }
        stage('Docker Image Scan') {
            steps {
                sh """docker run --rm -v ${WORKSPACE}:/${CI_PROJECT_NAME} -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy clean --all """
                sh """docker run --rm -v ${WORKSPACE}:/${CI_PROJECT_NAME} -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image --timeout 10m --format template --template "@contrib/html.tpl" --output ${CI_PROJECT_NAME}/${TRIVY_IMAGE_REPORT} $IMAGE_VERSION"""
                sh """curl -u ${NEXUS_USERNAME}:${NEXUS_PASSWORD} --upload-file ${TRIVY_IMAGE_REPORT} http://nexus.thienngo.tech/repository/trivy_report/${TRIVY_IMAGE_REPORT}"""
            }
        }
        stage('Push Docker Image') {
            steps {
                withDockerRegistry(credentialsId: 'harbor-cred', url: 'https://harbor.thienngo.click/') {
                    sh """docker tag thienngo/boardgame:${env.BUILD_ID} harbor.thienngo.click/boardgame/${IMAGE_VERSION}"""
                    sh """docker push harbor.thienngo.click/boardgame/${IMAGE_VERSION}"""

                }
            }
        }
        
    }
}

