pipeline {
    agent any

    environment {
        BACKUP_DIR = "/var/www/backups"
        PROD_DIR = "/var/www/produccion"
        LOG_FILE = "/var/log/jenkins/pipeline.log"
    }

    stages {
        stage('Preparar entorno') {
            steps {
                echo "Preparando entorno de trabajo..."
            }
        }
    }
}
