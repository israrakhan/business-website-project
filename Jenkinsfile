pipeline {
    agent any
    
    environment {
        APP_DIR = '/opt/checkout/chat-app'
        DEPLOY_DIR = '/opt/deployment/react'
        S3_BUCKET = 'your-s3-bucket-name'
        AWS_REGION = 'us-east-1'
    }
    
    stages {
        stage('Stop Running Deployment') {
            steps {
                script {
                    sh '''
                        pm2 stop chat-app || true
                        pm2 delete chat-app || true
                    '''
                }
            }
        }
        
        stage('Pull Fresh Code from Git') {
            steps {
                script {
                    sh '''
                        cd ${APP_DIR}
                        git pull origin main
                    '''
                }
            }
        }
        
        stage('Install Dependencies and Build') {
            steps {
                script {
                    sh '''
                        cd ${APP_DIR}
                        npm install
                        npm run build
                    '''
                }
            }
        }
        
        stage('Move Dist to Deployment Directory') {
            steps {
                script {
                    sh '''
                        rm -rf ${DEPLOY_DIR}/*
                        cp -r ${APP_DIR}/dist/* ${DEPLOY_DIR}/
                    '''
                }
            }
        }
        
        stage('Deploy using PM2') {
            steps {
                script {
                    sh '''
                        cd ${DEPLOY_DIR}
                        pm2 serve ${DEPLOY_DIR} 3000 --name "chat-app" --spa
                        pm2 save
                    '''
                }
            }
        }
        
        stage('Upload Dist to S3') {
            steps {
                script {
                    // Using S3 Publisher Plugin (s3Upload step)
                    s3Upload(
                        profileName: 'aws-credentials',
                        entries: [[
                            sourceFile: "${APP_DIR}/dist/**",
                            bucket: "${S3_BUCKET}",
                            selectedRegion: "${AWS_REGION}",
                            flatten: false,
                            managedArtifacts: true,
                            storageClass: 'STANDARD'
                        ]],
                        dontWaitForConcurrentBuildCompletion: false
                    )
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
            echo 'Application deployed and artifacts uploaded to S3.'
        }
        failure {
            echo 'Pipeline failed! Please check the logs.'
        }
        always {
            echo 'Pipeline execution completed.'
        }
    }
}
