pipeline {
    agent any

    tools {
        maven 'maven' // Ensure this matches the Maven installation name in Jenkins
    }

    environment {
        ImageName = 'my-app-image'
        BUILD_TAG = "latest"
        DOCKERHUB_USER = 'furqan28ca'
    }

    stages {
        stage('Checkout From Git') {
            steps {
                git branch: 'main', url: 'https://github.com/furqan28ca/spring-petclinic.git'
            }
        }

        stage('Maven Validate') {
            steps {
                echo 'Validating the project...'
                sh 'mvn validate'
            }
        }

        stage('Maven Compile') {
            steps {
                echo 'Compiling the project...'
                sh 'mvn compile'
            }
        }

        stage('Maven Test') {
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }

        stage('Maven Package') {
            steps {
                echo 'Packaging the project...'
                sh 'mvn package'
            }
        }

        stage('SonarCloud Analysis') {
            environment {
                SCANNER_HOME = tool 'sonar-scanner' // Matches tool config in Jenkins
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

        stage('Publish Sonar Report') {
            steps {
                echo 'Publishing SonarCloud report...'
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=sonarproject456_jenkins789 \
                        -Dsonar.organization=sonarproject456 \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=$SONAR_TOKEN \
                        -Dsonar.qualitygate.wait=false
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh '''
                    docker build -t ${ImageName}:${BUILD_TAG} .
                    docker tag ${ImageName}:${BUILD_TAG} ${DOCKERHUB_USER}/${ImageName}:${BUILD_TAG}
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                echo 'Running Trivy scan...'
                sh '''
                    trivy image --format table --severity HIGH,CRITICAL \
                        --output trivy-report.txt ${DOCKERHUB_USER}/${ImageName}:${BUILD_TAG}
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-report.txt'
                }
            }
        }

        stage('Login to Docker Hub and Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKERHUB_USER}/${ImageName}:${BUILD_TAG}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying to Kubernetes...'
                    sh '''
                        az aks get-credentials --resource-group demo11 --name lucky-aks-cluster11
                        kubectl apply -f k8s/petclinic.yml
                        kubectl get all
                    '''
                }
            }
        }
    }
}
