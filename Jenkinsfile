pipeline {
    agent any

    tools {
        maven 'Maven 3' 
    }

    stages {
        stage('1. Checkout') {
            steps {
                checkout scm
            }
        }
      
        stage('2. Build & Package') {
            steps {
                // Compila il progetto Jakarta EE generando il file WAR limitando i thread
                sh 'mvn clean package -T 2'
            }
        }

        stage('3. Deploy su Payara') {
            steps {
                script {
                    // Trova il file WAR generato nel workspace
                    def warFile = sh(script: "find . -path '*/target/*.war' | head -n 1", returnStdout: true).trim()
                    echo "Trovato WAR: ${warFile}"
                    
                    if (warFile) {
                        // 1. Crea il file password dentro il container proteggendo l'intero comando bash
                        sh "docker exec firstcup-payara bash -c \"echo 'AS_ADMIN_PASSWORD=' > /opt/payara/admin-password.txt\""
                        
                        // 2. Copia il WAR dal workspace di Jenkins direttamente dentro il container Payara
                        sh "docker cp ${warFile} firstcup-payara:/opt/payara/app.war"
                        
                        // 3. Esegue il deploy utilizzando il file password creato
                        sh "docker exec firstcup-payara /opt/payara/appserver/bin/asadmin --user admin --passwordfile=/opt/payara/admin-password.txt deploy --force=true /opt/payara/app.war"
                    } else {
                        error("Errore: Nessun file WAR trovato nel workspace!")
                    }
                }
            }
        }
    }

    post {
        success {
            echo '🎉 Pipeline completata con successo! Progetto compilato e deployato su Payara.'
        }
        failure {
            echo '❌ Build o Deploy fallito. Controlla i log della console.'
        }
    }
}
