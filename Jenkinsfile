pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/PSamu23/prjenkins.git'
        BACKUP_DIR = "/var/www/backups"
        PROD_DIR = "/var/www/produccion"
        TEST_DIR = "/var/www/pruebas"
        LOG_FILE = "/var/log/jenkins/pipeline.log"
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

        stage('Preparar entorno') {
            steps {
                echo 'Preparando entorno de trabajo...'
                sh 'mkdir -p ${BACKUP_DIR} ${PROD_DIR} ${TEST_DIR}'
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
                sh "echo 'Integración completada en desarrollo' >> ${LOG_FILE}"
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
                echo 'Desplegando en servidor de pruebas...'
                sh '''
                    cp -r ./ ${TEST_DIR}
                    echo "Despliegue completado en pruebas" >> ${LOG_FILE}
                '''
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
                echo 'Creando respaldo antes del despliegue...'
                sh '''
                    tar -czf ${BACKUP_DIR}/backup_$(date +%Y%m%d_%H%M%S).tar.gz ${PROD_DIR}
                    cp -r ./ ${PROD_DIR}
                    echo "Despliegue completado en producción" >> ${LOG_FILE}
                '''
            }
        }
    }

    post {
        failure {
            echo 'Pipeline fallido. Restaurando último respaldo...'
            sh '''
                LATEST_BACKUP=$(ls -t ${BACKUP_DIR} | head -n 1)
                if [ -n "$LATEST_BACKUP" ]; then
                    echo "Restaurando desde backup: $LATEST_BACKUP" >> ${LOG_FILE}
                    tar -xzf ${BACKUP_DIR}/$LATEST_BACKUP -C ${PROD_DIR}
                    echo "Restauración completada desde $LATEST_BACKUP" >> ${LOG_FILE}
                else
                    echo "No hay respaldos disponibles para restaurar" >> ${LOG_FILE}
                fi
            '''
        }
        success {
            echo 'Pipeline completado exitosamente.'
            sh "echo 'Pipeline ejecutado exitosamente' >> ${LOG_FILE}"
        }
    }
}
