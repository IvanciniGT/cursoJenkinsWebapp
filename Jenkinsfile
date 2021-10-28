// Plantilla de PIPELINE.
// Autor: IvanciniGT

// Al cambiar la verisón sólo se recarga la configuración del proyecto... pero no se ejecutarán las tareas
VERSION_DEL_PIPELINE="0"

// Añadir aquí los parámetros del pipeline
PARAMETROS_DE_MI_PIPELINE=[
    // booleanParam (defaultValue: true, description: 'Descripción de mi parámetro', name: 'MI_PARAM_BOOLEAN'),
    // string(defaultValue: 'valor por defecto', description: 'Descripción de mi parámetro de texto', name: 'MI_PARAM_TEXTO'),
    // choice(choices: ['Valor1', 'Valor2', 'Valor3'], description: 'Descripción de mi parámetro', name: 'MI_PARAM_LISTA')
]

// Añadir aquí los parámetros del pipeline
TRIGGERS_DE_MI_PIPELINE=[
    //cron("* * * * *"),                                                        // Ejecutar cada minuto
    //pollSCM("* * * * *")                                                      // Ejecutar cuando cambie algo en GIT
    //upstream(upstreamProjects: 'PROYECTO_DISPARADOR', threshold: 'FAILURE')   // Ejecutar tras otro proyecto (Incluso si falla)
]

// No tocar este trozo... es el que hace la magia
if (this.params.getOrDefault('VERSION_DEL_PIPELINE',"-1")!=VERSION_DEL_PIPELINE){
    stage('Configuración del JOB'){
        creoConfiguracion()
    }
    return
}
// Aquí empiezan las tareas propias de mi pipeline
node {
    checkout scm
    docker.image('maven:3.8.3-openjdk-8').inside {
        try{
            stage('Compilación') {
                sh 'mvn compile'
            }
            stage('Pruebas') {
                try{
                    sh 'mvn test'
                }
                finally{
                    echo 'Publico los resultados de los test'
                    junit 'target/surefire-reports/*.xml'
                }
            }
            stage('Empaquetado') {
                sh 'mvn package'
                echo 'Guardo el fichero WAR'
                archiveArtifacts artifacts: 'target/webapp.war', followSymlinks: false
            }
            stage('Despliegue') {
                echo 'Despliego el fichero WAR'
                
                
                deploy( adapters : [ tomcat9 (url: "http://172.31.3.123:8081", 
                                              credentialsId: "tomcat-user") ], 
                        war: "target/webapp.war",
                        contextPath: "miapp"
                )
                
                echo 'Lo pruebo, el despliegue'
                sh '''#!/bin/bash
                      sleep 5
                      curl http://172.31.3.123:8081/miapp/
                      curl -s http://172.31.3.123:8081/miapp/ | grep -c Hola
                       [[ $(curl -s http://172.31.3.123:8081/miapp/ | grep -c Hola ) != 1 ]] && exit 1
                   '''
                // JMETER
                echo 'Restauro el Tomcat'
            }
        }
        finally {
            stage ("Limpieza del workspace") {
                sh 'mvn clean'
            }
        }
    }
}












// Función que crea la configuración. NO TOCAR si no sabes lo que estás haciendo ;)
def creoConfiguracion(){
    properties(
        [
            parameters(
                [
                    choice(name: 'VERSION_DEL_PIPELINE', description: 'Version del pipeline instalada', choices: [ VERSION_DEL_PIPELINE ])
                ] + PARAMETROS_DE_MI_PIPELINE
            ),
            pipelineTriggers( TRIGGERS_DE_MI_PIPELINE )
        ]
    )
}
