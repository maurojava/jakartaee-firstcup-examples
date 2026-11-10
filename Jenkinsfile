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
                def warFile = sh(script: "find . -path '*/target/*.war' | head -n 1", returnStdout: true).trim()
                echo "Trovato WAR: ${warFile}"
                
                if (warFile) {
                    // Copia il file WAR nel container Payara
                    sh "docker cp ${warFile} firstcup-payara:/opt/payara/app.war"
                    
                    // Esegue il deploy con asadmin sfruttando l'accesso locale senza credenziali
                    sh "docker exec firstcup-payara /opt/payara/appserver/bin/asadmin deploy --force=true /opt/payara/app.war"
                } else {
                    error("Nessun file WAR trovato nel workspace!")
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
