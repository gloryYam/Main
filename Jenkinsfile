pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose-deploy.yml'
        MYSQL_ROOT_PASSWORD="${MYSQL_ROOT_PASSWORD}"
        MYSQL_DATABASE="${MYSQL_DATABASE}"
        MYSQL_USER="${MYSQL_USER}"
        MYSQL_PASSWORD="${MYSQL_PASSWORD}"
        DEPLOY_SERVER="ubuntu@54.180.212.28"
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                git branch: 'develop', url: 'https://github.com/gloryYam/ci-cd'
            }
        }

        stage('Test') {
            steps {
                script {
                    sh "docker --version"
                    sh "docker compose --version"
                    sh "echo hello world!"
                }
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                }
            }
            steps {
                script {
                    sshagent(['deploy-ssh']) {
                        sh """
                        ls
                        pwd
                        ssh ubuntu@54.180.212.28 'mkdir -p ~/Team-A' && scp -r -o StrictHostKeyChecking=no . ${DEPLOY_SERVER}:~/Team-A
                        ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER} '
                        cd ~ &&
                        ls -al &&
                        docker compose -f Team-A/${DOCKER_COMPOSE_FILE} down --remove-orphans &&
                        docker compose -f Team-A/${DOCKER_COMPOSE_FILE} pull &&
                        docker compose -f Team-A/${DOCKER_COMPOSE_FILE} up -d'
                        rm r -f Team-A
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs(cleanWhenNotBuilt: false,
                    deleteDirs: true,
                    disableDeferredWipeout: true,
                    notFailBuild: true,
                    patterns: [[pattern: '.gitignore', type: 'INCLUDE'],
                            [pattern: '.propsfile', type: 'EXCLUDE']])
        }
        success {
            echo 'Build and deployment successful!'
        }
        failure {
            echo 'Build or deployment failed.'
        }
    }
}