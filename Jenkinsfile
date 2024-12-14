pipeline {
    agent any

    environment {
        BACKUP_DIR = "/var/www/backups"
        PROD_DIR = "/var/www/produccion"
        TEST_DIR = "/var/www/pruebas"
        LOG_FILE = "/var/log/jenkins/pipeline.log"
    }

    stages {
        stage('Setup') {
            steps {
                echo "Configurando credenciales de Git..."
                script {
                    sh '''
                        git config --global user.email "samuelespinal90@gmail.com"
                        git config --global user.name "PSamu23"
                    '''
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Integrar a Desarrollo') {
            steps {
                echo "Integrando cambios en la rama desarrollo..."
                script {
                    sh '''
                    git checkout desarrollo
                    git pull origin desarrollo
                    '''
                }
                sh "echo 'Integración completada en desarrollo' >> ${LOG_FILE}"
            }
        }

        stage('Fusionar y Desplegar en Pruebas') {
            steps {
                echo "Fusionando cambios en pruebas y desplegando..."
                script {
                    sh '''
                    git checkout pruebas
                    git merge desarrollo
                    git push origin pruebas
                    '''
                }
                echo "Desplegando en servidor de pruebas..."
                
            }
        }

        stage('Fusionar y Desplegar en Producción') {
            steps {
                echo "Fusionando cambios en producción..."
                script {
                    sh '''
                    git checkout produccion
                    git merge pruebas
                    git push origin produccion
                    '''
                }
                echo "Creando respaldo antes del despliegue"
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
            echo "Pipeline fallido. Restaurando último respaldo..."
            script {
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
        }
        success {
            echo "Pipeline completado exitosamente."
            script {
                sh "echo 'Pipeline ejecutado exitosamente' >> ${LOG_FILE}"
            }
        }
    }
}
