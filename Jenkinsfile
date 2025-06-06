pipeline {
    agent any
    // Для ручного запуска с выбором окружения
    parameters {
        choice(name: 'ENV', choices: ['dev', 'prod'], description: 'Окружение')
    }
    triggers {
        // Для автоматического запуска по вебхуку
        githubPush()
    }
    options {
        // Разрешаем автоматический запуск без параметров
        skipDefaultCheckout(false)
    }
    environment {
        // Определяем окружение автоматически для триггеров вебхука
        AUTO_ENV = BRANCH_NAME == 'main' ? 'prod' : 'dev'
    }
    stages {
        stage('Checkout') {
            steps {
                // Всегда делаем чекаут репозитория
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
                                        // Пример команды для перезагрузки сервиса
                                        execCommand: "sudo systemctl restart ${targetEnv}-service"
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
            // Уведомление об успешном деплое
            slackSend channel: '#deployments', 
                      message: "✅ Deployed ${env.JOB_NAME} to ${params.ENV ?: env.AUTO_ENV}\nCommit: ${env.GIT_COMMIT}"
        }
        failure {
            // Уведомление об ошибке
            slackSend channel: '#errors', 
                      message: "❌ FAILED: ${env.JOB_NAME} to ${params.ENV ?: env.AUTO_ENV}\nBuild: ${env.BUILD_URL}"
        }
    }
}
