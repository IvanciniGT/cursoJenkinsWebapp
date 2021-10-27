// El primer elemento en un pipeline declarativo es pipeline
pipeline {
    
    // En que agente se va a ejecutar este job
    agent any;
    
    // Y otras cosas...
    
    // Etapas tiene mi pipeline. BIEN !!! PUEDO TENER VARIAS
    stages {
        stage('Compilación') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Pruebas') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    echo 'Publico los resultados de los test'
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Empaquetado') {
            steps {
                sh 'mvn package'
            }
            post {
                success {
                    echo 'Guardo el fichero WAR'
                    archiveArtifacts artifacts: 'target/webapp.war', followSymlinks: false
                }
            }
        }
        stage('Despliegue') {
            steps {
                echo 'Despliego el fichero WAR'
                echo 'Lo pruebo, el despliegue'
            }
            post {
                always {
                    echo 'Restauro el Tomcat'
                }
            }
        }
    }
    
    post {
        always {
            sh 'mvn clean'
        }
    }
}