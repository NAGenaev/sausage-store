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
        always {
            script {
                withCredentials([
                    string(credentialsId: 'TELEGRAM_BOT_TOKEN', variable: 'TOKEN'),
                    string(credentialsId: 'CHAT_ID', variable: 'CHAT_ID')
                ]) {
                    def buildStatus = currentBuild.currentResult
                    def buildTime = currentBuild.durationString
                    def triggeredBy = currentBuild.getBuildCauses()[0]?.userId ?: "Автоматически"
                    def gitBranch = env.GIT_BRANCH ?: "Неизвестно"
                    def gitCommit = env.GIT_COMMIT ?: "Неизвестно"

                    def message = """
📢 *Jenkins уведомление*  
${buildStatus == 'SUCCESS' ? '✅ *Сборка успешна!* 🎉' : '❌ *Ошибка сборки!* 🚨'}  

📦 *Проект:* ${env.JOB_NAME}  
🆔 *Build ID:* #${env.BUILD_NUMBER}  
🔗 [Ссылка на билд](${env.BUILD_URL})  

🏷 *Бранч:* ${gitBranch}  
🔀 *Коммит:* ${gitCommit}  
🖥 *Агент:* ${env.NODE_NAME}  
⏳ *Длительность:* ${buildTime}  
👤 *Запустил:* ${triggeredBy}  
                    """

                    sh """
                    curl -s -X POST "https://api.telegram.org/bot${TOKEN}/sendMessage" \\
                        -d "chat_id=${CHAT_ID}" \\
                        -d "text=${message}" \\
                        -d "parse_mode=Markdown"
                    """
                }
            }
        }
    }

} 