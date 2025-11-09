pipeline {
    agent any
    environment {
        DOCKERHUB_CRED = credentials('DockerHub')
        SONAR_TOKEN    = credentials('sonarqube-token')
        AZURE_SP       = credentials('azure-sp')
        AZURE_TENANT   = credentials('azure-tenant')
        GITHUB_TOKEN   = credentials('github-token')
        IMAGE_NAME     = "bhavananimmagadda/python-onepiece-app"
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/bhavananimmagadda-arch/python-onepiece-app.git'
            }
        }

        stage('Python Build') {
            steps {
                sh '''
                    set -e
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Python Test') {
            steps {
                sh '''
                    set -e
                    # Activate venv
                    . venv/bin/activate
                    # Run pytest if tests folder exists
                    if [ -d "tests" ]; then
                        pytest tests/
                    else
                        echo "No tests directory found, skipping tests."
                    fi
                '''
            }
        }

        stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('SonarScanner') {  // <- use your SonarScanner installation name
            sh """
                sonar-scanner \
                    -Dsonar.projectKey=jenkins-project \
                    -Dsonar.sources=. \
                    -Dsonar.python.version=3.12
            """
        }
    }
}

stage('Publish Quality Gate') {
    steps {
        timeout(time: 15, unit: 'MINUTES') {
            script {
                def qg = waitForQualityGate()
                if (qg.status != 'OK') {
                    error "Pipeline aborted due to failed Quality Gate: ${qg.status}"
                } else {
                    echo "Quality Gate passed: ${qg.status}"
                }
            }
        }
    }
}

        stage('Docker Build and Push') {
            steps {
                sh '''
                    set -e
                    echo $DOCKERHUB_CRED_PSW | docker login -u $DOCKERHUB_CRED_USR --password-stdin
                    docker build -t $IMAGE_NAME .
                    docker push $IMAGE_NAME
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    set -e
                    trivy image $IMAGE_NAME > trivy-report.txt || true
                '''
                archiveArtifacts artifacts: 'trivy-report.txt', fingerprint: true
            }
        }
    }
}
