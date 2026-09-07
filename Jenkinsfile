pipeline {
agent any

```
parameters {
    string(
        name: 'NGINX_HOST_PORT',
        defaultValue: '8084',
        description: 'Host port for Nginx'
    )

    string(
        name: 'CORS_ALLOWED_ORIGINS_RAW',
        defaultValue: 'http://localhost:8084',
        description: 'Allowed frontend origin'
    )
}

environment {
    MYSQL_ROOT_PASSWORD = credentials('task-mysql-root-password')
    MYSQL_PASSWORD = credentials('task-mysql-password')
    MYSQL_DB = 'taskdb'
    MYSQL_USER = 'appuser'
}

stages {

    stage('Checkout') {
        steps {
            echo 'Checking out latest code...'
            checkout scm
        }
    }

    stage('Check Docker') {
        steps {
            sh '''
                set -e
                docker --version
                docker compose version
            '''
        }
    }

    stage('Create Environment') {
        steps {
            sh '''
                set -e

                cat > .env <<EOF
```

MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
MYSQL_PASSWORD=${MYSQL_PASSWORD}
MYSQL_DB=${MYSQL_DB}
MYSQL_USER=${MYSQL_USER}
NGINX_HOST_PORT=${NGINX_HOST_PORT}
CORS_ALLOWED_ORIGINS_RAW=${CORS_ALLOWED_ORIGINS_RAW}
ENVIRONMENT=production
SESSION_LIFETIME_SECONDS=3600
EOF

```
                chmod 600 .env
            '''
        }
    }

    stage('Validate Compose') {
        steps {
            sh '''
                set -e
                docker compose config
                echo "Docker Compose configuration is valid."
            '''
        }
    }

    stage('Build Docker Images') {
        steps {
            echo 'Building Docker images...'
            sh '''
                set -e
                docker compose build
            '''
        }
    }

    stage('Deploy') {
        steps {
            echo 'Deploying Task Management App...'
            sh '''
                set -e
                docker compose up -d
            '''
        }
    }

    stage('Verify Deployment') {
        steps {
            sh '''
                set -e

                echo "Waiting for containers..."
                sleep 15

                docker compose ps
            '''
        }
    }

    stage('Health Check') {
        steps {
            sh '''
                set -e

                echo "Checking backend API..."

                curl --fail --silent --show-error \
                    http://localhost:${NGINX_HOST_PORT}/api/health

                echo ""
                echo "Backend health check passed."

                echo "Checking frontend..."

                curl --fail --silent --show-error \
                    -I http://localhost:${NGINX_HOST_PORT}

                echo ""
                echo "Frontend health check passed."
            '''
        }
    }
}

post {
    success {
        echo '=============================================='
        echo 'Task Management App deployed successfully!'
        echo '=============================================='
    }

    failure {
        echo '=============================================='
        echo 'Task Management App deployment FAILED!'
        echo '=============================================='

        sh '''
            docker compose ps || true
            docker compose logs --tail=50 || true
        '''
    }

    cleanup {
        script {
            if (fileExists('.env')) {
                sh 'rm -f .env'
                echo '.env removed from Jenkins workspace.'
            }
        }
    }
}
```

}

