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
                sh 'mvn clean package -T 2'
            }
        }

        stage('3. Deploy su Payara') {
            steps {
                script {
                    sh '''
                        # Trova il file WAR generato nel workspace
                        WAR_FILE=$(find . -path "*/target/*.war" | head -n 1)
                        
                        if [ -f "$WAR_FILE" ]; then
                            echo "Trovato WAR: $WAR_FILE. Copia in corso nel container Payara..."
                            
                            # Copia il WAR dal workspace di Jenkins direttamente dentro il container Payara
                            docker cp "$WAR_FILE" firstcup-payara:/opt/payara/app.war
                            
                            echo "Esecuzione deploy con asadmin..."
                            docker exec firstcup-payara /opt/payara/appserver/bin/asadmin --user admin deploy --force=true /opt/payara/app.war
                        else
                            echo "Errore: Nessun file WAR trovato nel workspace!"
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
        }
    }
}
