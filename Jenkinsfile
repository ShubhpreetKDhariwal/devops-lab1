pipeline {
    agent any

    environment {
        S3_BUCKET = 'devops-lab1-artifacts-1'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Lint') {
            steps {
                sh '''
                    venv/bin/flake8 app.py
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    venv/bin/pytest test_app.py -v
                '''
            }
        }

        stage('Package') {
            steps {
                sh '''
                    COMMIT_SHA=$(git rev-parse --short HEAD)

                    mkdir -p artifact
                    cp app.py artifact/

                    tar -czf devops-lab1-${COMMIT_SHA}.tar.gz artifact/

                    echo $COMMIT_SHA > commit_sha.txt
                '''
            }
        }

        stage('Upload to S3') {
            steps {
                sh '''
                    COMMIT_SHA=$(cat commit_sha.txt)

                    aws s3 cp \
                    devops-lab1-${COMMIT_SHA}.tar.gz \
                    s3://${S3_BUCKET}/devops-lab1-${COMMIT_SHA}.tar.gz
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Artifact was not published.'
        }
    }
}