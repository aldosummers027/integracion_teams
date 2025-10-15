// Jenkinsfile (Versión simplificada para solo crear el artefacto)
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

    // --- ACCIONES POST-EJECUCIÓN SIMPLIFICADAS ---
    // Este bloque se ejecuta siempre al final para archivar y limpiar.
    post {
        always {
            node('agente-ansible') {
                echo "Pipeline finalizado. Archivando el reporte..."

                // 1. Archiva el reporte CSV para que sea descargable desde la página del build.
                archiveArtifacts artifacts: 'reporte_disponibilidad_*.csv', allowEmptyArchive: true
            }
        }
    }
}