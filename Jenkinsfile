pipeline {
    agent any // Выбираем Jenkins агента, на котором будет происходить сборка: нам нужен любой

    triggers {
        pollSCM('H/5 * * * *') // Запускать будем автоматически по крону примерно раз в 5 минут
    }

    tools {
        maven 'Maven' // Для сборки бэкенда нужен Maven
        jdk 'JDK 16' // И Java Developer Kit нужной версии
        nodejs 'NodeJS 8' // А NodeJS нужен для фронта
    }

    stages {
        stage('Build & Test backend') {
            steps {
                dir("backend") { // Переходим в папку backend
                    sh 'mvn package' // Собираем мавеном бэкенд
                }
            }

            post {
                success {
                    junit 'backend/target/surefire-reports/**/*.xml' // Передадим результаты тестов в Jenkins
                }
            }
        }

        stage('Build frontend') {
            steps {
                dir("frontend") {
                    sh 'npm install' // Для фронта сначала загрузим все сторонние зависимости
                    sh 'npm run build' // Запустим сборку
                }
            }
        }
        
        stage('Save artifacts') {
            steps {
                archiveArtifacts(artifacts: 'backend/target/sausage-store-0.0.1-SNAPSHOT.jar')
                archiveArtifacts(artifacts: 'frontend/dist/frontend/*')
            }
        }

    }

    post {
        success {
            script {
                withCredentials([
                    string(credentialsId: 'TELEGRAM_BOT_TOKEN', variable: 'TOKEN'),
                    string(credentialsId: 'CHAT_ID', variable: 'CHAT_ID')
                ]) {
                    def MESSAGE = "✅ *Сборка успешна!* 🎉\n" +
                                  "📦 *Проект:* ${env.JOB_NAME}\n" +
                                  "🆔 *Build ID:* #${env.BUILD_NUMBER}\n" +
                                  "🔗 [Перейти в Jenkins](${env.BUILD_URL})"

                    sh """
                    curl -s -X POST "https://api.telegram.org/bot${TOKEN}/sendMessage" \\
                        -d "chat_id=${CHAT_ID}" \\
                        -d "text=${MESSAGE}" \\
                        -d "parse_mode=Markdown"
                    """
                }
            }
        }
        failure {
            script {
                withCredentials([
                    string(credentialsId: 'TELEGRAM_BOT_TOKEN', variable: 'TOKEN'),
                    string(credentialsId: 'TELEGRAM_CHAT_ID', variable: 'CHAT_ID')
                ]) {
                    def MESSAGE = "❌ *Сборка провалилась!* 😢\n" +
                                  "📦 *Проект:* ${env.JOB_NAME}\n" +
                                  "🆔 *Build ID:* #${env.BUILD_NUMBER}\n" +
                                  "🔗 [Перейти в Jenkins](${env.BUILD_URL})"

                    sh """
                    curl -s -X POST "https://api.telegram.org/bot${TOKEN}/sendMessage" \\
                        -d "chat_id=${CHAT_ID}" \\
                        -d "text=${MESSAGE}" \\
                        -d "parse_mode=Markdown"
                    """
                }
            }
        }
    }

} 