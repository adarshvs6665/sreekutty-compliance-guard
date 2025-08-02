pipeline {
    agent any
    parameters {
        choice(name: 'APP_NAME', choices: ['vulnerable-app', 'secure-app'], description: 'Select app to deploy')
    }
    tools {
        nodejs 'nodejs24'
    }
    environment {
        PATH = "${env.PATH}:${env.HOME}/.local/bin"
        BUCKET_NAME = "secure-bucket"
    }
    stages {
        stage('Pull main') {
            steps {
                git branch: 'main', url: 'https://github.com/adarshvs6665/sreekutty-compliance-guard.git'
            }
        }
        stage('Install dependencies') {
            steps {
                dir("${params.APP_NAME}") {
                    sh 'npm ci'
                }
                sh '''
                    if ! command -v checkov &> /dev/null; then
                        pip3 install --user checkov
                    fi

                    if ! command -v sam &> /dev/null; then
                        pip3 install aws-sam-cli
                    fi
                    sam --version
                '''
            }
        }
        stage('Run checks in parallel') {
            parallel {
                stage('Run lint') {
                    steps {
                        dir("${params.APP_NAME}") {
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
                            def template = params.APP_NAME == 'secure-app' ? 'template2.yaml' : 'template.yaml'
                            def command = params.APP_NAME == 'secure-app' ? 
                                "checkov -f ${template} --config-file .checkov.yaml" : 
                                "checkov -f ./${template}"
                            try {
                                sh command
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
                withAWS(credentials: 'aws-creds', region: 'eu-west-1') {
                    script {
                        def template = params.APP_NAME == 'secure-app' ? 'template2.yaml' : 'template.yaml'
                        sh 'sam build'
                        sh "sam deploy --template-file ${template} --no-confirm-changeset --capabilities CAPABILITY_IAM --region eu-west-1"
                    }
                }
            }
        }
    }
    post {
        failure {
            echo 'Pipeline failed due to a compliance violation.'
        }
        success {
            echo 'Deployment succeeded!'
        }
    }
}