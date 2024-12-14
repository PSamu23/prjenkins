pipeline {
    agent any

    environment {
        // Ajusta el ID de credenciales configurado en Jenkins
        GIT_CREDENTIALS_ID = 'your-credentials-id'
        REPO_URL = 'https://github.com/PSamu23/prjenkins.git'
    }

    stages {
        stage('Declarative: Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/main']], 
                    userRemoteConfigs: [[
                        url: REPO_URL,
                        credentialsId: GIT_CREDENTIALS_ID
                    ]]
                ])
            }
        }

        stage('Setup') {
            steps {
                echo 'Configurando credenciales de Git...'
                sh '''
                    git config --global user.email "samuelespinal90@gmail.com"
                    git config --global user.name "PSamu23"
                '''
            }
        }

        stage('Sync Ramas Locales') {
            steps {
                script {
                    sh '''
                        git fetch origin
                        git checkout pruebas || git checkout -b pruebas
                        git reset --hard origin/pruebas

                        git checkout desarrollo || git checkout -b desarrollo
                        git reset --hard origin/desarrollo

                        git checkout main
                        git reset --hard origin/main
                    '''
                }
            }
        }

        stage('Integrar a Desarrollo') {
            steps {
                echo 'Integrando cambios en la rama desarrollo...'
                script {
                    sh '''
                        git checkout desarrollo
                        git pull origin desarrollo
                    '''
                }
            }
        }

        stage('Fusionar y Desplegar en Pruebas') {
            steps {
                echo 'Fusionando cambios en pruebas y desplegando...'
                script {
                    withCredentials([usernamePassword(credentialsId: GIT_CREDENTIALS_ID, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh '''
                            git checkout pruebas
                            git merge desarrollo
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/PSamu23/prjenkins.git pruebas
                        '''
                    }
                }
            }
        }

        stage('Fusionar y Desplegar en Producción') {
            when {
                expression {
                    currentBuild.result == null || currentBuild.result == 'SUCCESS'
                }
            }
            steps {
                echo 'Fusionando cambios en producción y desplegando...'
                script {
                    withCredentials([usernamePassword(credentialsId: GIT_CREDENTIALS_ID, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh '''
                            git checkout main
                            git merge pruebas
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/PSamu23/prjenkins.git main
                        '''
                    }
                }
            }
        }
    }

    post {
        failure {
            echo 'Pipeline fallido. Restaurando último respaldo...'
            script {
                sh '''
                    if [ -d /var/www/backups ] && [ "$(ls -A /var/www/backups)" ]; then
                        echo "Restaurando respaldo..."
                        # Comando para restaurar respaldo (ajustar según necesidad)
                    else
                        echo "No hay respaldos disponibles para restaurar"
                    fi
                '''
            }
        }
    }
}
