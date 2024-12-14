pipeline {
    agent any

    environment {
        // Configuración de credenciales y repositorio
        REPO_URL = 'https://github.com/PSamu23/prjenkins.git'
    }

    stages {
        stage('Declarative: Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/main']], 
                    userRemoteConfigs: [[
                        url: REPO_URL,
                        credentialsId: 'github-token-psamu23111'
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

                        # Asegurarse de que existan las ramas locales
                        git checkout pruebas || git checkout -b pruebas
                        git reset --hard origin/pruebas || echo "Rama pruebas no sincronizada"

                        git checkout desarrollo || git checkout -b desarrollo
                        git reset --hard origin/desarrollo || echo "Rama desarrollo no sincronizada"

                        git checkout main
                        git reset --hard origin/main || echo "Rama main no sincronizada"
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
                        git pull origin desarrollo || echo "Error al actualizar desarrollo"
                    '''
                }
            }
        }

        stage('Fusionar y Desplegar en Pruebas') {
            steps {
                echo 'Fusionando cambios en pruebas y desplegando...'
                script {
                    withCredentials([string(credentialsId: 'github-token-psamu23111', variable: 'GIT_TOKEN')]) {
                        sh '''
                            git checkout pruebas
                            git merge desarrollo || echo "Conflictos al fusionar desarrollo en pruebas"
                            git push https://${GIT_TOKEN}@github.com/PSamu23/prjenkins.git pruebas
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
                    withCredentials([string(credentialsId: 'github-token-psamu23111', variable: 'GIT_TOKEN')]) {
                        sh '''
                            git checkout main
                            git merge pruebas || echo "Conflictos al fusionar pruebas en main"
                            git push https://${GIT_TOKEN}@github.com/PSamu23/prjenkins.git main
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
                        # Comando para restaurar respaldo
                    else
                        echo "No hay respaldos disponibles para restaurar"
                    fi
                '''
            }
        }
    }
}
