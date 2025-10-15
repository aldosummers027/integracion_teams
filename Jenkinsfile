// Jenkinsfile
pipeline {
    agent any

    parameters {
        string(
            name: 'TARGET_VM_NAME', 
            defaultValue: '', 
            description: 'Escribe el nombre exacto de la VM para generar el reporte de disponibilidad.'
        )
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
                echo "Código descargado correctamente."
            }
        }

        stage('Ejecutar Reporte de Disponibilidad') {
            steps {
                // ▼▼ USAREMOS ESTE BLOQUE DE CREDENCIALES ▼▼
                withCredentials([
                    string(credentialsId: 'azure-client-id', variable: 'AZURE_CLIENT_ID'),
                    string(credentialsId: 'azure-client-secret', variable: 'AZURE_CLIENT_SECRET'),
                    string(credentialsId: 'azure-tenant-id', variable: 'AZURE_TENANT_ID')
                ]) {
                    script {
                        echo "Ejecutando playbook para la VM: ${params.TARGET_VM_NAME}"
                        sh """
                            source /home/ubuntu/ansible_venv/bin/activate
                            ansible-playbook disponibilidad_vm_especifica.yml -e "target_vm_name=${params.TARGET_VM_NAME}"
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finalizado. Archivando los reportes generados..."
            archiveArtifacts artifacts: 'reporte_disponibilidad_*.csv', allowEmptyArchive: true
            cleanWs()
        }
    }
}