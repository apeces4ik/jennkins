pipeline {
    agent any
    parameters {
        choice(name: 'ENV', choices: ['dev', 'prod'], description: 'Окружение')
    }
    stages {
        stage('Deploy') {
            steps {
                echo "Deploying to ${params.ENV}"
                cleanWs()
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'my-ssh-server',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: '**/*',
                                    remoteDirectory: "/var/www/${params.ENV}"
                                )
                            ]
                        )
                    ]
                )
            }
        }
    }
}
pipeline {
    agent any
    parameters {
        choice(
            name: 'ENV',
            choices: ['dev', 'prod'],
            description: 'Выбор окружения для деплоя'
        )
    }
    stages {
        // Ваши этапы
    }
}
