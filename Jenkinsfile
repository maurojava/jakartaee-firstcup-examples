pipeline {
    agent any

    tools {
        // Assicurati che questo nome corrisponda a quello in Manage Jenkins -> Tools
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
                // Compila il codice ed esegue i test producendo il file .war
                sh 'mvn clean package'
            }
        }

        stage('3. Deploy su Payara') {
            steps {
                script {
                    // Copia o esegue il deploy del file WAR direttamente nel container Payara
                    sh '''
                        WAR_FILE=$(find firstcup-war/target -name "*.war" | head -n 1)
                        if [ -f "$WAR_FILE" ]; then
                            echo "Deploying $WAR_FILE to Payara..."
                            docker exec firstcup-payara /opt/payara/appserver/bin/asadmin --user admin --passwordfile=/opt/payara/passwordfile deploy --force=true $WAR_FILE
                        else
                            echo "File WAR non trovato in firstcup-war/target!"
                            exit 1
                        fi
                    '''
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
            /* Decommenta se hai configurato l'SMTP in Jenkins:
            emailext (
                subject: "❌ [Jenkins] Build Fallita: ${env.JOB_NAME} - #${env.BUILD_NUMBER}",
                body: "La pipeline è fallita. Analizza i log: ${env.BUILD_URL}",
                to: 'maurokube@gmail.com'
            )
            */
        }
    }
}

