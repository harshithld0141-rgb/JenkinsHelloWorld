pipeline {
    agent any

    // ─── Poll SCM: check GitHub every 5 minutes ───────────────────────────────
    triggers {
        pollSCM('H/5 * * * *')   // cron: every 5 min  |  change to 'H * * * *' for hourly
    }

    // ─── Global environment variables ─────────────────────────────────────────
    environment {
        GITHUB_REPO      = 'https://github.com/harshithld0141-rgb/JenkinsHelloWorld.git'
        GITHUB_BRANCH    = 'master'
        DOCKER_IMAGE     = 'myapp'                        // local image name
        DOCKER_TAG       = "${env.BUILD_NUMBER}"          // unique tag per build
        DOCKER_LATEST    = 'latest'
    }

    stages {

        // ── 1. Checkout ────────────────────────────────────────────────────────
        stage('Checkout') {
            steps {
                echo "📥 Pulling code from GitHub branch: ${env.GITHUB_BRANCH}"
                git branch: "${env.GITHUB_BRANCH}",
                    url: "${env.GITHUB_REPO}"
                    // credentialsId: 'github-credentials'   // uncomment for private repos
            }
        }

        // ── 2. Build Docker Image ──────────────────────────────────────────────
        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image: ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                sh """
                    docker build \
                        --no-cache \
                        -t ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} \
                        -t ${env.DOCKER_IMAGE}:${env.DOCKER_LATEST} \
                        .
                """
            }
        }


        // ── 4. Deploy locally on same instance ────────────────────────────────
        stage('Deploy') {
            steps {
                echo "🟢 Deploying container on this instance..."
                sh """
                    # Stop & remove old container if running
                    docker stop myapp-container 2>/dev/null || true
                    docker rm   myapp-container 2>/dev/null || true

                    # Run new container
                    docker run -d \
                        --name myapp-container \
                        --restart unless-stopped \
                        -p 9090:8080 \
                        ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
                """
            }
        }
    }

    // ─── Post-build actions ────────────────────────────────────────────────────
    post {
        always {
            echo "🧹 Cleaning up dangling Docker images..."
            sh 'docker image prune -f'
        }
        success {
            echo "✅ Pipeline completed successfully! Image: ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}"
        }
        failure {
            echo "❌ Pipeline FAILED. Check logs above for details."
        }
    }
}
