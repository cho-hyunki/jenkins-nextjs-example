pipeline {
    agent any
    
    // GitHub 트리거 설정 추가
    triggers {
        githubPush()
    }
    
    environment {
        NODE_VERSION = '20.11.1'  // LTS version
        DEPLOY_PATH = '/var/www/nextjs-app'  // 배포할 경로
        // GitHub 관련 환경변수 추가
        GITHUB_TOKEN = credentials('github-token')
    }
    
    stages {
        stage('Setup') {
            steps {
                // Clean workspace
                cleanWs()
                
                // Checkout code
                checkout scm
                
                // Setup Node.js
                sh 'curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash'
                sh '. ~/.nvm/nvm.sh && nvm install ${NODE_VERSION} && nvm use ${NODE_VERSION}'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh '. ~/.nvm/nvm.sh && npm ci'
            }
        }
        
        stage('Lint') {
            steps {
                sh '. ~/.nvm/nvm.sh && npm run lint'
            }
        }
        
        stage('Build') {
            steps {
                sh '. ~/.nvm/nvm.sh && npm run build'
            }
        }
        
        stage('Test') {
            steps {
                sh '. ~/.nvm/nvm.sh && npm test'
            }
        }

        stage('Deploy') {
            steps {
                // 배포 디렉토리 생성
                sh 'mkdir -p ${DEPLOY_PATH}'
                
                // 빌드 결과물 복사
                sh 'cp -r .next ${DEPLOY_PATH}/'
                sh 'cp -r public ${DEPLOY_PATH}/'
                sh 'cp package*.json ${DEPLOY_PATH}/'
                
                // 배포 서버에서 의존성 설치 및 서버 재시작
                dir('${DEPLOY_PATH}') {
                    sh '. ~/.nvm/nvm.sh && npm ci --production'
                    sh 'pm2 restart nextjs-app || pm2 start npm --name "nextjs-app" -- start'
                }
            }
        }
    }
    
    post {
        always {
            // Clean workspace after build
            cleanWs()
        }
        success {
            // GitHub commit status 업데이트
            githubSetCommitStatus(
                context: 'Jenkins Pipeline',
                state: 'SUCCESS',
                message: 'Build succeeded!'
            )
        }
        failure {
            // GitHub commit status 업데이트
            githubSetCommitStatus(
                context: 'Jenkins Pipeline',
                state: 'FAILURE',
                message: 'Build failed!'
            )
        }
    }
} 