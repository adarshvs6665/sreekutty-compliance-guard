pipeline {
    agent any
    tools {
        nodejs 'NodeJS 24.x'
    }
    environment {
        SAM_TEMPLATE = 'template.yaml'
    }
    stages {
        stage('Pull main') {
            steps {
                git branch: 'main', url: 'https://github.com/adarshvs6665/sreekutty-compliance-guard.git'
            }
        }
        stage('Run lint') {
            steps {
                dir('demo-app') {
                    sh 'npm ci'
                    sh 'npm run lint'
                }
            }
        }
        stage('Run checkov') {
            steps {
                sh 'checkov -f template.yaml --quiet --soft-fail false'
            }
        }
        stage('SAM deploy') {
            steps {
                sh 'sam build'
                sh 'sam deploy --no-confirm-changeset --capabilities CAPABILITY_IAM'
            }
        }
    }
    post {
        failure {
            echo 'Pipeline failed due to a policy violation.'
        }
        success {
            echo 'Deployment succeeded!'
        }
    }
}
