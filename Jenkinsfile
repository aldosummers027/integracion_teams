// Jenkinsfile (Versión corregida y limpia)
pipeline {
    // Define el agente donde se ejecutará el pipeline.
    agent any

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
                // Inyecta las credenciales de Azure de forma segura.
                // Asegúrate de tener una credencial en Jenkins de tipo "Microsoft Azure Service Principal"
                // con el ID 'azure-credentials-prod' (o el que hayas configurado).
                withCredentials([azureServicePrincipal('azure-credentials-prod')]) {
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
    } // <-- El bloque 'stages' termina aquí

    // --- ACCIONES POST-EJECUCIÓN ---
    // Este bloque se ejecuta siempre al final.
    post {
        always {
            echo "Pipeline finalizado. Archivando los reportes generados..."
            
            // Archiva el reporte CSV para que sea descargable.
            archiveArtifacts artifacts: 'reporte_disponibilidad_*.csv', allowEmptyArchive: true
            
            // Limpia el workspace.
            cleanWs()
        }
    }
}