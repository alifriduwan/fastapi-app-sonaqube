pipeline {
    agent {
        docker {
            image 'python:3.11'
            args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        SONARQUBE = credentials('sonarqube-token-fastapi')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/alifriduwan/fastapi-app-sonaqube'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    pip install --upgrade pip
                    pip install -r requirements.txt || true
                    pip install fastapi uvicorn pytest pytest-cov coverage requests httpx
                '''
            }
        }

        stage('Run Tests & Coverage') {
            steps {
                sh 'pytest --cov=app --cov-report=xml --cov-report=term tests/'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-25') {
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=FastAPI-app \
                          -Dsonar.sources=./app \
                          -Dsonar.python.coverage.reportPaths=coverage.xml \
                          -Dsonar.host.url=http://host.docker.internal:9001/ \
                          -Dsonar.login=$SONARQUBE
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t fastapi-app:latest .'
            }
        }

        stage('Deploy Container') {
            steps {
                sh 'docker run -d -p 8000:8000 fastapi-app:latest'
            }
        }
    }

    post {
        always {
            echo "Pipeline finished"
        }
    }
}
