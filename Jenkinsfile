// Jenkinsfile (Versión final con limpieza de workspace)
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
                // ▼▼▼ PASO AÑADIDO: LIMPIAR EL WORKSPACE PRIMERO ▼▼▼
                cleanWs()

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
            post {
                success {
                    echo "Guardando el reporte generado para pasarlo a la etapa final."
                    stash name: 'reporte-generado', includes: 'reporte_disponibilidad_*.csv'
                }
            }
        }
    }

    // --- ACCIONES POST-EJECUCIÓN ---
    post {
        always {
            node('agente-ansible') {
                echo "Pipeline finalizado. Archivando el reporte..."
                unstash 'reporte-generado'

                // 1. Archiva el reporte CSV.
                archiveArtifacts artifacts: 'reporte_disponibilidad_*.csv', allowEmptyArchive: true

                // 2. Limpia el workspace al final (opcional, pero buena práctica).
                //    Se movió la limpieza principal al inicio del stage.
                cleanWs()
            }
        }
    }
}