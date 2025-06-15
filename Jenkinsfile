/* import shared library */
@Library('jenkins-shared-library')_

pipeline {
    agent none
    stages {
        stage('test golang') {
            agent { docker { image 'golangci/golangci-lint:latest' } }
            steps {
                script { checkgolang }
            }
        }
        stage('Check docker-compose syntax') {
            agent { docker { image 'docker/compose' } }
            steps {
                sh 'docker-compose -f \${WORKSPACE}/docker-compose.yml config'
            }
        }
        stage('Check Dockerfile syntax') {
            agent { docker { image 'hadolint/hadolint' } }
            steps {
                sh 'hadolint --ignore DL3006 --ignore DL3022 \${WORKSPACE}/Dockerfile'
            }
        }
    }
    post {
    always {
       script {
         /* Use slackNotifier.groovy from shared library and provide current build result as parameter */
         clean
         slackNotifier currentBuild.result
     }
    }
    }
    }
