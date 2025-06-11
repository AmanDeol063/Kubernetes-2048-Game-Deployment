pipeline {
    agent any

    tools {
        nodejs 'nodejs-22-6-0'
    }

    environment {
        DOCKER_IMAGE = 'amandeol063/2048-game'
        SONARQUBE_ENV = 'SonarQube'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'npx sonar-scanner'
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                sh '''
                mkdir -p reports
                dependency-check.sh --project 2048-game --scan . --out reports
                '''
            }
        }

        stage('Build React App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$GIT_COMMIT .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                trivy image $DOCKER_IMAGE:$GIT_COMMIT \
                  --severity CRITICAL \
                  --exit-code 0 \
                  --format json -o trivy-critical.json
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_IMAGE:$GIT_COMMIT
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-id', variable: 'KUBECONFIG')]) {
                    sh '''
                    kubectl apply -f k8s-yaml/namespace.yaml
                    kubectl apply -f k8s-yaml/deployment.yaml
                    kubectl apply -f k8s-yaml/service.yaml
                    kubectl apply -f k8s-yaml/ingress.yaml
                    '''
                }
            }
        }
    }

    post {
        always {
            script {
                sh '''
                trivy convert --format template --template "/usr/local/share/trivy/templates/html.tpl" \
                  --output trivy-report.html trivy-critical.json || true
                '''
            }

            publishHTML([
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: '.',
                reportFiles: 'trivy-report.html',
                reportName: 'Trivy Vulnerability Report'
            ])

            publishHTML([
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'reports',
                reportFiles: 'dependency-check-report.html',
                reportName: 'OWASP Dependency Check Report'
            ])
        }

        success {
            echo '🎉 2048 Game deployed successfully on Kubernetes!'
        }

        failure {
            echo '💥 Build failed. Check logs above.'
        }
    }
}
