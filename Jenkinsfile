pipeline {
    agent any

    environment {
        APP_NAME = "cicd-web-demo"
        STAGING_PORT = "8081"
        PROD_PORT = "8082"
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage("Checkout") {
            steps {
                echo "Descargando el codigo..."
            }
        }

        stage("Lint / Validacion") {
            steps {
                echo "Validando estructura minima..."
                sh "test -f Dockerfile"
                sh "test -f docker-compose.yml"
                sh "test -f app/index.html"
                sh "test -x scripts/test.sh"
                echo "Validacion OK"
            }
        }

        stage("Test") {
            steps {
                echo "Ejecutando pruebas..."
                sh "./scripts/test.sh"
            }
        }

        stage("Build Imagen (staging)") {
            steps {
                echo "Construyendo imagen para staging..."
                sh "docker build -t :staging ."
            }
        }

        stage("Deploy a Staging") {
            steps {
                echo "Desplegando en STAGING (puerto )..."
                sh "docker compose up -d web-staging"
                echo "Staging actualizado. Verifica en: http://IP-VM:8081"
            }
        }

        stage("Aprobacion para Produccion") {
            steps {
                input message: "Aprobar despliegue a PRODUCCION?", ok: "Si, desplegar"
            }
        }

        stage("Promover Imagen a Produccion") {
            steps {
                echo "Promoviendo imagen a produccion..."
                sh "docker tag :staging :production"
            }
        }

        stage("Deploy a Produccion") {
            steps {
                echo "Desplegando en PRODUCCION (puerto )..."
                sh "docker compose up -d web-production"
                echo "Produccion actualizada. Verifica en: http://IP-VM:8082"
            }
        }
    }

    post {
        success {
            echo "CI/CD completado con exito."
        }
        failure {
            echo "CI/CD fallo. Revisar logs del build."
        }
        always {
            sh "docker ps --format \"table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}\" || true"
        }
    }
}
