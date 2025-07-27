pipeline {
    agent any
    tools {
        nodejs 'nodejs24'
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
        stage('Install dependencies') {
            steps {
                dir('demo-app') {
                    sh 'npm ci'
                }
                sh '''
                    if ! command -v checkov &> /dev/null; then
                        pip install --user checkov
                        export PATH=$PATH:$HOME/.local/bin
                    fi
                '''
            }
        }
        stage('Run checks in parallel') {
            parallel {
                stage('Run lint') {
                    steps {
                        dir('demo-app') {
                            script {
                                try {
                                    sh 'npm run lint'
                                } catch (err) {
                                    currentBuild.result = 'FAILURE'
                                    echo 'Lint failed'
                                    throw err
                                }
                            }
                        }
                    }
                }
                stage('Run checkov') {
                    steps {
                        script {
                            try {
                                sh '''
                                    export PATH=$PATH:$HOME/.local/bin
                                    checkov -f template.yml
                                '''
                            } catch (err) {
                                currentBuild.result = 'FAILURE'
                                echo 'Checkov failed'
                                throw err
                            }
                        }
                    }
                }
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
