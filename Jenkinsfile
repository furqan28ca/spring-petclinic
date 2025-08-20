pipeline {
    agent any

    tools {
        maven 'maven' // Ensure this matches Maven installation in Jenkins
    }

    environment {
        ImageName = 'my-app-image'
        BUILD_TAG = "latest"
    }

    stages {
        stage('Checkout From Git') {
            steps {
                git branch: 'main', url: 'https://github.com/furqan28ca/spring-petclinic.git'
            }
        }

        stage('Maven Build') {
            steps {
                echo 'Building project with Maven...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarCloud Analysis') {
            environment {
                SCANNER_HOME = tool 'sonar-scanner'
            }
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.organization=sonarproject456 \
                        -Dsonar.projectName=jenkins \
                        -Dsonar.projectKey=sonarproject456_jenkins789 \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.host.url=https://sonarcloud.io
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh '''
                    docker build -t ${ImageName}:${BUILD_TAG} .
                    docker tag ${ImageName}:${BUILD_TAG} luckyregistryy.azurecr.io/${ImageName}:${BUILD_TAG}
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                echo 'Running Trivy scan...'
                sh '''
                    trivy image --format table --severity HIGH,CRITICAL \
                        --output trivy-report.txt luckyregistryy.azurecr.io/${ImageName}:${BUILD_TAG}
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-report.txt'
                }
            }
        }

        stage('Login to ACR and Push Image') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'azure-sp', usernameVariable: 'AZURE_USERNAME', passwordVariable: 'AZURE_PASSWORD'),
                    string(credentialsId: 'azure-tenant', variable: 'TENANT_ID')
                ]) {
                    script {
                        echo "Logging into Azure Container Registry..."
                        sh '''
                            az login --service-principal -u "$AZURE_USERNAME" -p "$AZURE_PASSWORD" --tenant "$TENANT_ID"
                            az acr login --name luckyregistryy
                            docker push luckyregistryy.azurecr.io/${ImageName}:${BUILD_TAG}
                        '''
                    }
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                script {
                    echo 'Deploying to AKS...'
                    sh '''
                        az aks get-credentials --resource-group demo11 --name lucky-aks-cluster11
                        kubectl apply -f k8s/deployment.yaml
                        kubectl get pods
                    '''
                }
            }
        }
    }
}
