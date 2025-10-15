// Jenkinsfile (Versión corregida con stash/unstash)
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
            // ▼▼▼ PASO AÑADIDO: GUARDAR EL REPORTE ▼▼▼
            // Este bloque post se ejecuta solo si la etapa fue exitosa.
            post {
                success {
                    echo "Guardando el reporte generado para pasarlo a la etapa final."
                    stash name: 'reporte-generado', includes: 'reporte_disponibilidad_*.csv'
                }
            }
        }
    }

    // --- ACCIONES POST-EJECUCIÓN SIMPLIFICADAS ---
    post {
        always {
            node('agente-ansible') {
                echo "Pipeline finalizado. Archivando el reporte..."

                // ▼▼▼ PASO AÑADIDO: RECUPERAR EL REPORTE ▼▼▼
                echo "Recuperando el reporte guardado."
                unstash 'reporte-generado'

                // 1. Archiva el reporte CSV. Ahora sí lo encontrará.
                archiveArtifacts artifacts: 'reporte_disponibilidad_*.csv', allowEmptyArchive: true

                // 2. Limpia el workspace para la siguiente ejecución.
                cleanWs()
            }
        }
    }
}