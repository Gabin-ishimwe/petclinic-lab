pipeline {
    agent any

    environment {
        DOCKER_REPO = 'gabinishimwe/petclinic'
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials')
    }

    stages {
        stage('Checkout') {
            steps {
                echo "=== CHECKOUT STAGE ==="
                echo "Repository: ${env.GIT_URL}"
                echo "Branch: ${env.GIT_BRANCH}"
                checkout scm
                sh '''
                    echo "Repository contents:"
                    ls -la
                    pwd
                '''
            }
        }

        stage('Build') {
            steps {
                echo "=== BUILD STAGE ==="
                sh '''
                    echo "Java version:"
                    java -version
                    chmod +x ./mvnw
                    echo "Maven version:"
                    ./mvnw --version
                    echo "Building Spring Boot application..."
                    ./mvnw clean compile -DskipTests
                '''
            }
        }

        stage('Test') {
            steps {
                echo "=== TEST STAGE ==="
                sh '''
                    chmod +x ./mvnw
                    echo "Running tests..."
                    ./mvnw test
                '''
            }
//             post {
//                 always {
//                     publishTestResults testResultsPattern: 'target/surefire-reports/*.xml'
//                 }
//             }
        }

//         stage('Static Analysis') {
//             steps {
//                 echo "=== STATIC ANALYSIS STAGE ==="
//                 sh '''
//                     chmod +x ./mvnw
//                     echo "Running basic static analysis..."
//                     ./mvnw compile
//                     echo "Java files count:"
//                     find src -name "*.java" | wc -l
//                 '''
//             }
//         }

        stage('Package') {
            steps {
                echo "=== PACKAGE STAGE ==="
                sh '''
                    chmod +x ./mvnw
                    echo "Packaging application..."
                    ./mvnw package -DskipTests
                    echo "Generated artifacts:"
                    ls -la target/
                '''
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "=== DOCKER BUILD STAGE ==="
                sh '''
                    echo "Docker version:"
                    docker --version
                    echo "Building Docker image..."
                    docker build -t ${DOCKER_REPO}:${BUILD_NUMBER} .
                    docker tag ${DOCKER_REPO}:${BUILD_NUMBER} ${DOCKER_REPO}:latest
                    echo "Docker images:"
                    docker images | grep petclinic
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "=== DOCKER PUSH STAGE ==="
                sh '''
                    echo "Logging into Docker Hub..."
                    echo ${DOCKER_CREDENTIALS_PSW} | docker login -u ${DOCKER_CREDENTIALS_USR} --password-stdin
                    echo "Pushing images..."
                    docker push ${DOCKER_REPO}:${BUILD_NUMBER}
                    docker push ${DOCKER_REPO}:latest
                    echo "✅ Images pushed successfully!"
                '''
            }
        }
    }

    post {
        always {
            echo "=== CLEANUP ==="
            sh '''
                docker system prune -f || echo "Docker cleanup completed"
            '''
            cleanWs()
        }
        success {
            echo "🎉 PIPELINE SUCCESS!"
            echo "✅ Image available: ${DOCKER_REPO}:${BUILD_NUMBER}"
            echo "🔗 Docker Hub: https://hub.docker.com/r/gabinishimwe/petclinic"
        }
        failure {
            echo "❌ PIPELINE FAILED!"
            echo "Check the logs above for error details"
        }
    }
}
