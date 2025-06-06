pipeline {
    agent any
    parameters {
        choice(name: 'ENV', choices: ['dev', 'prod'], description: 'Окружение')
    }
    triggers {
        githubPush()
    }
    options {
        skipDefaultCheckout(false)
    }
    environment {
        AUTO_ENV = "dev" // значение по умолчанию
    }
    stages {
        stage('Initialize') {
            steps {
                script {
                    // Получаем имя ветки если не установлено
                    env.BRANCH_NAME = env.BRANCH_NAME ?: sh(
                        script: 'git rev-parse --abbrev-ref HEAD',
                        returnStdout: true
                    ).trim()
                    
                    // Обновляем AUTO_ENV на основе ветки
                    env.AUTO_ENV = (env.BRANCH_NAME == 'main') ? 'prod' : 'dev'
                }
            }
        }
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Deploy') {
            steps {
                script {
                    def targetEnv = params.ENV ?: env.AUTO_ENV
                    echo "Deploying to ${targetEnv}"
                    deleteDir()
                    
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'my-ssh-server',
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: '**/*',
                                        remoteDirectory: "/var/www/${targetEnv}",
                                        execCommand: """
                                            echo 'Successfully deployed to ${targetEnv}'
                                            systemctl restart ${targetEnv}-service
                                        """
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
    post {
        success {
            slackSend channel: '#deployments',
                      message: "✅ Deployed ${env.JOB_NAME} to ${targetEnv}"
        }
        failure {
            slackSend channel: '#errors',
                      message: "❌ FAILED: ${env.JOB_NAME} to ${targetEnv}"
        }
    }
}
