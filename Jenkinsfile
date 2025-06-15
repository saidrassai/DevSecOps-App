/* import shared library */
@Library('jenkins-shared-library')_

pipeline {
    agent any
    
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'preprod', 'prod'],
            description: 'Select deployment environment'
        )
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Skip test execution'
        )
    }
    
    environment {
        DOCKER_IMAGE = "devsecops-app"
        DOCKER_TAG = "${BUILD_NUMBER}-${ENVIRONMENT}"
        REGISTRY = "your-registry" // Update with your Docker registry
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()
                }
            }
        }
        
        stage('Code Quality & Security') {
            parallel {
                stage('Golang Lint') {
                    agent { docker { image 'golangci/golangci-lint:latest' } }
                    steps {
                        script { checkgolang }
                    }
                }
                stage('Docker Compose Syntax') {
                    agent { docker { image 'docker/compose' } }
                    steps {
                        sh 'docker-compose -f ${WORKSPACE}/docker-compose.yml config'
                    }
                }
                stage('Dockerfile Lint') {
                    agent { docker { image 'hadolint/hadolint' } }
                    steps {
                        sh 'hadolint --ignore DL3006 --ignore DL3022 ${WORKSPACE}/Dockerfile'
                    }
                }
                stage('Security Scan') {
                    agent { docker { image 'securecodewarrior/docker-security-scanning' } }
                    steps {
                        script {
                            // Add security scanning tools like Trivy, Snyk, etc.
                            sh 'echo "Running security scans..."'
                            // sh 'trivy fs --security-checks vuln .'
                        }
                    }
                }
            }
        }
        
        stage('Unit Tests') {
            when {
                not { params.SKIP_TESTS }
            }
            agent { docker { image 'golang:1.19' } }
            steps {
                sh '''
                    go mod tidy
                    go test -v ./...
                    go test -coverprofile=coverage.out ./...
                '''
            }
            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: '.',
                        reportFiles: 'coverage.out',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest-${ENVIRONMENT}
                    """
                }
            }
        }
        
        stage('Image Security Scan') {
            steps {
                script {
                    sh """
                        echo "Scanning Docker image for vulnerabilities..."
                        # docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
        
        stage('Deploy to Environment') {
            steps {
                script {
                    switch(params.ENVIRONMENT) {
                        case 'dev':
                            deployToEnvironment('dev')
                            break
                        case 'preprod':
                            deployToEnvironment('preprod')
                            break
                        case 'prod':
                            input message: 'Deploy to Production?', ok: 'Deploy'
                            deployToEnvironment('prod')
                            break
                    }
                }
            }
        }
        
        stage('Post-Deployment Tests') {
            when {
                expression { params.ENVIRONMENT != 'prod' }
            }
            steps {
                script {
                    sh """
                        echo "Running post-deployment tests for ${ENVIRONMENT}..."
                        # Add your integration tests here
                        # curl -f http://your-${ENVIRONMENT}-url/health || exit 1
                    """
                }
            }
        }
        
        stage('Promote to Next Environment') {
            when {
                allOf {
                    expression { params.ENVIRONMENT == 'dev' }
                    expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
                }
            }
            steps {
                script {
                    input message: 'Promote to Pre-Production?', ok: 'Promote'
                    build job: env.JOB_NAME, 
                          parameters: [
                              choice(name: 'ENVIRONMENT', value: 'preprod'),
                              booleanParam(name: 'SKIP_TESTS', value: true)
                          ]
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Cleanup
                sh """
                    docker rmi -f ${DOCKER_IMAGE}:${DOCKER_TAG} || true
                    docker system prune -f
                """
                
                // Notifications
                clean
                slackNotifier currentBuild.result
            }
        }
        success {
            echo "Pipeline completed successfully for ${ENVIRONMENT} environment!"
        }
        failure {
            echo "Pipeline failed for ${ENVIRONMENT} environment!"
            // Send alerts to teams
        }
    }
}

def deployToEnvironment(environment) {
    echo "Deploying to ${environment} environment..."
    
    // Use Ansible for deployment
    sh """
        ansible-playbook -i inventory/${environment} \
                        -e docker_image=${DOCKER_IMAGE} \
                        -e docker_tag=${DOCKER_TAG} \
                        -e environment=${environment} \
                        playbooks/deploy.yml
    """
    
    // Alternative: Direct Docker deployment
    // sh """
    //     docker-compose -f docker-compose.${environment}.yml up -d
    // """
}
