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
        // Правильное объявление переменных окружения
        AUTO_ENV = "${BRANCH_NAME == 'main' ? 'prod' : 'dev'}"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Определяем целевое окружение
                    def targetEnv = params.ENV ?: env.AUTO_ENV
                    echo "Deploying to ${targetEnv}"
                    
                    // Очищаем рабочую директорию
                    deleteDir()
                    
                    // Деплоим через SSH
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'my-ssh-server',
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: '**/*',
                                        remoteDirectory: "/var/www/${targetEnv}",
                                        execCommand: "echo 'Deployed to ${targetEnv}'"
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}
