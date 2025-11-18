pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: maven
    image: maven:3.8.4-openjdk-11
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /root/.m2
      name: maven-cache
  - name: docker
    image: docker:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true
  volumes:
  - name: maven-cache
    emptyDir: {}
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
"""
        }
    }

    environment {
        DOCKER_HUB_REPO = 'gabinishimwe/petclinic'  // Update with your Docker Hub username
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-credentials')
        BUILD_VERSION = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out repository..."
                checkout scm
                sh 'ls -la'
            }
        }

        stage('Build') {
            steps {
                container('maven') {
                    echo "🔨 Building Spring Boot application..."
                    sh '''
                        mvn clean compile -DskipTests
                        echo "Build completed successfully"
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                container('maven') {
                    echo "🧪 Running tests..."
                    sh '''
                        mvn test
                        echo "Tests completed"
                    '''
                }
            }
            post {
                always {
                    publishTestResults testResultsPattern: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Static Analysis') {
            steps {
                container('maven') {
                    echo "🔍 Running static analysis..."
                    sh '''
                        # Basic static analysis - you can enhance this
                        mvn compile
                        echo "Static analysis completed"
                    '''
                }
            }
        }

        stage('Package') {
            steps {
                container('maven') {
                    echo "Packaging application..."
                    sh '''
                        mvn package -DskipTests
                        ls -la target/*.jar
                    '''
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                container('docker') {
                    echo "Building Docker image..."
                    sh '''
                        docker build -t ${DOCKER_HUB_REPO}:${BUILD_VERSION} .
                        docker tag ${DOCKER_HUB_REPO}:${BUILD_VERSION} ${DOCKER_HUB_REPO}:latest
                        echo "Docker image built successfully"
                    '''
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                container('docker') {
                    echo "Pushing to Docker Hub..."
                    sh '''
                        echo ${DOCKER_HUB_CREDENTIALS_PSW} | docker login -u ${DOCKER_HUB_CREDENTIALS_USR} --password-stdin
                        docker push ${DOCKER_HUB_REPO}:${BUILD_VERSION}
                        docker push ${DOCKER_HUB_REPO}:latest
                        echo "Images pushed successfully to Docker Hub"
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo "✅ Pipeline completed successfully!"
            echo "Image available: ${DOCKER_HUB_REPO}:${BUILD_VERSION}"
        }
        failure {
            echo "❌ Pipeline failed - check logs for details"
        }
    }
}
