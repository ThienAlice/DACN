
@Library('jenkins-libraries') _
pipeline {
    agent { label 'build-server' }
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
                    dirs.each { dir ->
                        echo "Running pipeline for: ${dir}"
                        if (dir == 'store-front') {
                            build job: "../store-front-service", parameters: 
                            [
                                string(name: 'GIT_COMMIT', value: env.GIT_COMMIT),
                                string(name: 'BRANCH_NAME', value: env.BRANCH_NAME),
                            ]
                        } else if (dir == 'order-service') {
                            build job: "../order-service", parameters: 
                            [
                                string(name: 'GIT_COMMIT', value: env.GIT_COMMIT),
                                string(name: 'BRANCH_NAME', value: env.BRANCH_NAME),
                            ]
                        }
                        else if (dir == 'product-service') {
                            build job: "../product-service", parameters: 
                            [
                                string(name: 'GIT_COMMIT', value: env.GIT_COMMIT),
                                string(name: 'BRANCH_NAME', value: env.BRANCH_NAME),
                            ]
                        }
                    }
                }
            }
        }       
    }
    post {
        always {
            cleanWs()
        }
    }
}
