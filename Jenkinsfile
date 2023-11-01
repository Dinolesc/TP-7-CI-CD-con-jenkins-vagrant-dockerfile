pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = 'tpn7kickthemout'  // Nombre de tu imagen Docker
        DOCKER_HUB_REPO = 'dino08/tpn6-kickthemout'  // Cambia esto por tu DockerHub username/repo
        DOCKER_IMAGE_TAG = sh(script: 'date +%Y%m%d%H%M%S', returnStdout: true).trim()
        CONTAINER_NAME = 'TPN7.5'  // Nombre de tu contenedor Docker
        
    }

     stages {
          stage('Construir y etiquetar imagen') {
            steps {
                script {
                    sh 'docker --version'  // Comando de prueba de Docker
                    // Construir la imagen Docker y taggearla
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
                }
            }
        }

        stage('Crear contenedor') {
            steps {
                script {
                    // Ejecutar el contenedor con la imagen recién construida
                    sh "docker run -d --name ${CONTAINER_NAME} ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('Pruebas de la aplicación') {
            steps {
                script {
                    // Verificar que el contenedor esté en ejecución
                    def containerStatus = sh(returnStatus: true, script: 'docker ps -q -f name=TPN7.5')

                    if (containerStatus == 0) {
                        currentBuild.result = 'SUCCESS'
                        echo 'La prueba de integridad pasó: el contenedor está en ejecución.'
                    } else {
                        currentBuild.result = 'FAILURE'
                        error 'La prueba de integridad falló: el contenedor no se encuentra en ejecución.'
                }
            }
        }
    }      

        stage('Subir imagen a DockerHub') {
            steps {
                script {
                    // Iniciar sesión en DockerHub
                    withCredentials([usernamePassword(credentialsId: '2', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh "docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
                    }

                    // Subir la imagen a DockerHub
                    sh "docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${DOCKER_HUB_REPO}:${DOCKER_IMAGE_TAG}"
                    sh "docker push ${DOCKER_HUB_REPO}:${DOCKER_IMAGE_TAG}"

                    // Limpiar
                    sh "docker logout"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline ejecutado exitosamente."
        }
        failure {
            echo "Error en el pipeline."
        }
    }
}
