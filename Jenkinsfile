pipeline {
    agent any
    environment {
        DOCKERHUB_CRED = credentials('DockerHub')
        SONAR_TOKEN = credentials('sonarqube-token')
        AZURE_SP = credentials('azure-sp')
        AZURE_TENANT = credentials('azure-tenant')
        IMAGE_NAME = "your-dockerhub-username/python-onepiece-app"
    }
   stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/yourusername/your-repo.git'
            }
        } 
        stage('Python Build') {
            steps {
                sh 'python3 -m pip install -r requirements.txt'
            }
        }
        stage('Python Test') {
            steps {
                sh 'pytest tests/'
            }
        }
        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonarqube-local') {
                    sh "sonar-scanner -Dsonar.projectKey=jenkins-project -Dsonar.sources=. -Dsonar.host.url=http://<VM_PUBLIC_IP>:9000 -Dsonar.login=$SONAR_TOKEN"
                }
            }
        }
        stage('Publish Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Docker Build and Push') {
            steps {
                sh "docker login -u $DOCKERHUB_CRED_USR -p $DOCKERHUB_CRED_PSW"
                sh "docker build -t $IMAGE_NAME ."
                sh "docker push $IMAGE_NAME"
            }
        }
        stage('Trivy Scan') {
            steps {
                sh "trivy image $IMAGE_NAME > trivy-report.txt || true"
                archiveArtifacts artifacts: 'trivy-report.txt', fingerprint: true
            }
        }
    }
}