// Jenkinsfile (Versión Completa y Final)
pipeline {
    // Define el agente donde se ejecutará el pipeline.
    agent any

    environment {
        // Inyecta la URL del webhook desde una credencial de Jenkins.
        // Debes crear una credencial de tipo "Secret text" con el ID 'power-automate-webhook-url'
        // y pegar ahí la URL que te dio el segundo flujo de Power Automate.
        POWER_AUTOMATE_WEBHOOK_URL = credentials('power-automate-webhook-url')
    }

    // --- SECCIÓN DE PARÁMETROS ---
    parameters {
        string(
            name: 'TARGET_VM_NAME',
            defaultValue: '',
            description: 'Escribe el nombre exacto de la VM para generar el reporte de disponibilidad.'
        )
    }

    // --- ETAPAS DEL PIPELINE ---
    stages {
        // Etapa 1: Descargar el código desde Git
        stage('Checkout SCM') {
            steps {
                checkout scm
                echo "Código descargado correctamente."
            }
        }

        // Etapa 2: Ejecutar el Playbook de Ansible
        stage('Ejecutar Reporte de Disponibilidad') {
            steps {
                // Usamos el método de credenciales separadas que ya te funcionó.
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

    // --- ACCIONES POST-EJECUCIÓN ---
    // Este bloque se ejecuta siempre al final, sin importar si el pipeline falló o tuvo éxito.
    post {
        always {
            echo "Pipeline finalizado. Archivando y notificando..."

            // 1. Archiva el reporte CSV para que sea descargable desde Jenkins.
            archiveArtifacts artifacts: 'reporte_disponibilidad_*.csv', allowEmptyArchive: true

            // 2. Notifica a Power Automate que el job terminó y el artefacto está listo.
            script {
                echo "Enviando notificación a Power Automate..."
                sh '''
                    # Construimos el nombre del artefacto dinámicamente.
                    # Esto asegura que coincida con el archivo que se acaba de crear.
                    ARTIFACT_NAME="reporte_disponibilidad_${params.TARGET_VM_NAME}.csv"

                    # Enviamos los datos (Build Number, Nombre del Artefacto y Nombre del Job) a la URL del webhook.
                    # Estos datos son los que Power Automate necesita para construir la URL de descarga.
                    curl -X POST -H "Content-Type: application/json" \
                    -d '{"buildNumber": "${BUILD_NUMBER}", "artifactName": "'$ARTIFACT_NAME'", "jobName": "${JOB_NAME}"}' \
                    "${POWER_AUTOMATE_WEBHOOK_URL}"
                '''
            }

            // 3. Limpia el workspace para la siguiente ejecución.
            cleanWs()
        }
    }
}