// Jenkinsfile (Versión con 'node' en el post)
pipeline {
    agent any

    environment {
        // Inyecta la URL del webhook desde una credencial de Jenkins.
        POWER_AUTOMATE_WEBHOOK_URL = credentials('power-automate-webhook-url')
    }

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

    // --- ACCIONES POST-EJECUCIÓN CORREGIDAS ---
    post {
        always {
            // Se asigna un 'node' para darle contexto de workspace a los pasos.
            node('agente-ansible') {
                echo "Pipeline finalizado. Archivando y notificando..."

                // 1. Archiva el reporte CSV.
                archiveArtifacts artifacts: 'reporte_disponibilidad_*.csv', allowEmptyArchive: true

                // 2. Notifica a Power Automate.
                script {
                    echo "Enviando notificación a Power Automate..."
                    sh '''
                        ARTIFACT_NAME="reporte_disponibilidad_${params.TARGET_VM_NAME}.csv"
                        curl -X POST -H "Content-Type: application/json" \
                        -d '{"buildNumber": "${BUILD_NUMBER}", "artifactName": "'$ARTIFACT_NAME'", "jobName": "${JOB_NAME}"}' \
                        "${POWER_AUTOMATE_WEBHOOK_URL}"
                    '''
                }

                // 3. Limpia el workspace.
                cleanWs()
            }
        }
    }
}