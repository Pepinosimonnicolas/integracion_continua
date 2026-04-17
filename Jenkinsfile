pipeline {
    agent any

    environment {
        REGISTRY = "registry.br-ainiac.net"
        IMAGE_NAME = "${REGISTRY}/hello-world"
        STACK_NAME = "prueba-hello"
    }

    stages {
        stage('Limpieza inicial') {
            steps {
                echo 'Limpiando espacio...'
                sh "docker image prune -f"
            }
        }

        stage('Construir Imagen') {
            steps {
                script {
                    // Creamos un index.html básico para el Hello World
                    sh "echo '<h1>Hola prueba CI/CD Prueba de Swarm + Traefik Exitosa+commit GIT :)</h1>' > index.html"
                    // Dockerfile rápido en línea para la prueba
                    sh "echo 'FROM nginx:alpine\nCOPY index.html /usr/share/nginx/html/' > Dockerfile"
                    
                    sh "docker build -t ${IMAGE_NAME}:${env.BUILD_ID} ."
                    sh "docker tag ${IMAGE_NAME}:${env.BUILD_ID} ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Subir al Registro') {
            steps {
                echo "Subiendo imagen al registro local..."
                sh "docker push ${IMAGE_NAME}:${env.BUILD_ID}"
                sh "docker push ${IMAGE_NAME}:latest"
            }
        }

        stage('Desplegar en Swarm') {
            steps {
                echo "Eliminando stack antiguo..."
                sh "docker stack rm ${STACK_NAME} || true"
                    
                // 2. Esperamos unos segundos para que Docker libere los recursos
                echo "Esperando que los contenedores se detengan..."
                sleep 5
                echo "Desplegando stack en el clúster..."
                // El flag --with-registry-auth permite que los workers descarguen la imagen del registro privado
                sh "docker stack deploy -c docker-compose.yml ${STACK_NAME} --with-registry-auth"
            }
        }
    }

    post {
        success {
            echo "¡Todo listo! Revisa https://hello.br-ainiac.net"
        }
        failure {
            echo "Algo falló. Revisa los logs de Jenkins."
        }
    }
}
