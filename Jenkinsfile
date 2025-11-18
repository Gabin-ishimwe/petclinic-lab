pipeline {
    agent any

    environment {
        DOCKER_REPO = 'gabinishimwe/petclinic'
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials')
    }

    stages {
        stage('Setup Docker CLI') {
            steps {
                echo "=== INSTALLING DOCKER CLI ==="
                sh '''
                    # Install Docker CLI in workspace
                    if ! command -v docker &> /dev/null; then
                        echo "Installing Docker CLI..."
                        curl -fsSL https://download.docker.com/linux/static/stable/aarch64/docker-24.0.7.tgz | tar -xz
                        chmod +x docker/docker
                        echo "Docker CLI ready in workspace"
                    fi

                    # Test Docker
                    ./docker/docker --version
                '''
            }
        }
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
                    ./mvnw clean compile -DskipTests -B -Denforcer.skip=true -Dcheckstyle.skip=true
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

        stage('Docker Build & Push') {
            steps {
                echo "=== DOCKER BUILD & PUSH ==="
                sh '''
                    # Use local Docker CLI
                    DOCKER_CMD="./docker/docker"

                    echo "Building image..."
                    $DOCKER_CMD build -t ${DOCKER_REPO}:${BUILD_NUMBER} .
                    $DOCKER_CMD tag ${DOCKER_REPO}:${BUILD_NUMBER} ${DOCKER_REPO}:latest

                    echo "Pushing to Docker Hub..."
                    echo ${DOCKER_CREDENTIALS_PSW} | $DOCKER_CMD login -u ${DOCKER_CREDENTIALS_USR} --password-stdin
                    $DOCKER_CMD push ${DOCKER_REPO}:${BUILD_NUMBER}
                    $DOCKER_CMD push ${DOCKER_REPO}:latest

                    echo "Success!"
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
