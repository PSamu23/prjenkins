pipeline {
    agent any

    environment {
        BACKUP_DIR = "/var/www/backups"      // Directorio para respaldos
        PROD_DIR = "/var/www/produccion"    // Directorio de producción
        LOG_FILE = "/var/log/jenkins/pipeline.log" // Archivo de log
    }

    stages {
        stage('Integrar a Desarrollo') {
            steps {
                echo "Integrando cambios en la rama desarrollo..."
                script {
                    sh '''
                    git checkout desarrollo
                    git pull origin desarrollo
                    '''
                }
            }
        }

        stage('Pruebas') {
            steps {
                echo "Fusionando cambios en pruebas y desplegando en servidor de pruebas..."
                script {
                    sh '''
                    git checkout pruebas
                    git merge desarrollo
                    git push origin pruebas
                    '''
                }
                sh 'cp -r ./ /var/www/pruebas/'  // Copia los cambios al servidor de pruebas
            }
        }

        stage('Desplegar a Producción') {
            steps {
                echo "Fusionando cambios en producción..."
                script {
                    sh '''
                    git checkout produccion
                    git merge pruebas
                    git push origin produccion
                    '''
                }
                echo "Creando respaldo antes del despliegue en producción..."
                sh '''
                mkdir -p ${BACKUP_DIR}
                tar -czf ${BACKUP_DIR}/backup_$(date +%Y%m%d_%H%M%S).tar.gz ${PROD_DIR}
                cp -r ./ ${PROD_DIR}
                '''
            }
        }
    }

    post {
        failure {
            echo "Pipeline fallido. Restaurando último respaldo..."
            sh '''
            LATEST_BACKUP=$(ls -t ${BACKUP_DIR} | head -n 1)
            if [ -n "$LATEST_BACKUP" ]; then
                echo "Restaurando desde backup: $LATEST_BACKUP"
                tar -xzf ${BACKUP_DIR}/$LATEST_BACKUP -C ${PROD_DIR}
            else
                echo "No hay respaldos disponibles para restaurar"
            fi
            '''
        }
    }
}
