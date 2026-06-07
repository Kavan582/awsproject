pipeline {
    agent any
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        AWS_ACCOUNT_ID = '831492333688'
        AWS_REGION = 'us-east-1'
        ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/amazon-prime"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', 
                url: 'https://github.com/Kavan582/awsproject'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=amazon-prime \
                        -Dsonar.projectKey=amazon-prime"""
                }
            }
        }
        stage('NPM Install') {
            steps {
                sh 'npm install'
            }
        }
        stage('Trivy Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-report.html .'
            }
        }
        stage('Docker Build') {
            steps {
                sh "docker build -t amazon-prime ."
            }
        }
        stage('Push to ECR') {
            steps {
                withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS \
                        --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                        
                        docker tag amazon-prime:latest ${ECR_REPO}:latest
                        docker push ${ECR_REPO}:latest
                    """
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline completed!'
        }
    }
}
