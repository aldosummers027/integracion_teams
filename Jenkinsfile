// Jenkinsfile
pipeline {
    // Define dónde se ejecutará el pipeline. 'any' significa cualquier agente disponible.
    agent any

    // --- SECCIÓN DE PARÁMETROS ---
    // Aquí definimos los valores que Jenkins le pedirá al usuario al iniciar el job.
    parameters {
        string(
            name: 'TARGET_VM_NAME', 
            defaultValue: '', 
            description: 'Escribe el nombre exacto de la VM para generar el reporte de disponibilidad.'
        )
    }

    // --- COMIENZAN LAS ETAPAS DEL PIPELINE ---
    stages {
        // Etapa 1: Descargar el código desde Git
        stage('Checkout SCM') {
            steps {
                // Clona el repositorio donde se encuentra este Jenkinsfile
                checkout scm
                echo "Código descargado correctamente."
            }
        }

        // Etapa 2: Ejecutar el Playbook de Ansible
        stage('Ejecutar Reporte de Disponibilidad') {
            steps {
                // El bloque 'withCredentials' inyecta de forma segura las credenciales de Azure
                // como variables de entorno. Ansible las detectará automáticamente.
                withCredentials([azureServicePrincipal('azure-credentials-prod')]) {
                    script {
                        echo "Ejecutando playbook para la VM: ${params.TARGET_VM_NAME}"
                        
                        // NOTA: Se asume que 'ansible-playbook' está instalado en el agente de Jenkins.
                        // El comando -e (extra-vars) pasa el parámetro del usuario al playbook.
                        sh '''
                            source /home/ubuntu/ansible_venv/bin/activate
                            ansible-playbook playbook_reporte_dinamico.yml -e "target_vm_name=${params.TARGET_VM_NAME}"
                        '''
                    }
                }
            }
        }
    }

    // --- ACCIONES POST-EJECUCIÓN ---
    // Este bloque se ejecuta siempre al final, sin importar si el pipeline falló o tuvo éxito.
    post {
        always {
            echo "Pipeline finalizado. Archivando los reportes generados..."
            
            // Busca cualquier archivo CSV que empiece con "reporte_disponibilidad_"
            // y lo guarda como un "artefacto" del build.
            // Esto hará que el archivo sea descargable desde la página del job en Jenkins.
            archiveArtifacts artifacts: 'reporte_disponibilidad_*.csv', allowEmptyArchive: true
            
            // Limpia el workspace para no dejar archivos residuales.
            cleanWs()
        }
    }
}