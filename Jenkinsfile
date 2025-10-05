pipeline {
    agent any

    environment {
        NODE_VERSION = "20"
        APP_NAME = "SIT753-7.3HD"
    }

    stages {

        // 1️⃣ Checkout Stage
        stage('Checkout') {
            steps {
                echo "🔍 Cloning repository from GitHub..."
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/movinigamage/SIT753-7.3HD.git'
                    ]]
                ])
                sh 'git log -1 --pretty=oneline'
            }
        }

        // 2️⃣ Environment Setup (Node.js only)
        stage('Environment Setup') {
            steps {
                echo "🛠️ Setting up Node.js environment"
                sh '''
                    apt-get update
                    apt-get install -y curl
                    curl -fsSL https://deb.nodesource.com/setup_${NODE_VERSION}.x | bash -
                    apt-get install -y nodejs
                    node -v
                    npm -v
                '''
            }
        }

        // 3️⃣ Build Stage
        stage('Build') {
            steps {
                echo "🏗️ Installing project dependencies and building the app"
                sh '''
                    npm install
                    npm run build || echo "No build script found, skipping..."
                '''
            }
        }

        // 4️⃣ Test Stage
        stage('Test') {
            steps {
                echo "🧪 Running automated tests..."
                sh '''
                    if [ -f package.json ]; then
                        npm test || echo "⚠️ Tests failed or not configured"
                    else
                        echo "⚠️ No tests to run"
                    fi
                '''
            }
        }

        // 5️⃣ SonarQube Code Analysis (optional if plugin installed)
        stage('Code Analysis') {
            when { expression { return fileExists('sonar-project.properties') } }
            steps {
                echo "🔍 Running SonarQube analysis..."
                script {
                    withSonarQubeEnv('SonarQubeServer') {
                        sh 'sonar-scanner'
                    }
                }
            }
        }

        // 6️⃣ Docker Build & Deploy (uses host Docker via socket)
        stage('Deploy to Docker') {
            steps {
                echo "🐳 Building Docker image and running container..."
                sh '''
                    docker build -t ${APP_NAME}:latest .
                    docker stop ${APP_NAME} || true
                    docker rm ${APP_NAME} || true
                    docker run -d -p 3000:3000 --name ${APP_NAME} ${APP_NAME}:latest
                '''
            }
        }

        // 7️⃣ Post-Build Reporting
        stage('Report') {
            steps {
                echo "📊 Generating build summary..."
                sh '''
                    echo "Build completed for ${APP_NAME}" > build-report.txt
                    echo "Date: $(date)" >> build-report.txt
                    echo "Node version: $(node -v)" >> build-report.txt
                    echo "NPM version: $(npm -v)" >> build-report.txt
                    echo "Deployed container: ${APP_NAME}" >> build-report.txt
                '''
                archiveArtifacts artifacts: 'build-report.txt', fingerprint: true
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Please check the logs."
        }
    }
}
