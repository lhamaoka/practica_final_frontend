pipeline{

  agent {
      kubernetes {
        yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: lhamaoka/nodo-nodejs-practica-final
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-socket-volume
    securityContext:
      privileged: true
  volumes:
  - name: docker-socket-volume
    hostPath:
      path: /var/run/docker.sock
      type: Socket
    command:
    - sleep
    args:
    - infinity
        '''
        defaultContainer 'shell'
      }
  }

  stages {
    stage("1.- Prepare environment") {
        steps {
            sh "node -v"
        }
    }

    stage("2.- Build") {
        steps {
            sh "echo Consistirá en construir la aplicación Angular"
        }
    }

    stage("3.- Quality Tests") {
        steps {
            sh "echo Comprobación de la calidad del código con Sonarqube."
        }
    }

    stage("4.- Build & Push") {
        steps {
            sh "echo Construcción de la imagen con Kaniko y subida de la misma a vuestro repositorio personal en Docker Hub"
        }
    }

    stage("5.- Run test environment") {
        steps {
            sh "echo Iniciar un pod o contenedor con la imagen con la imagen que acabamos de generar"
        }
    }

    stage("6.- Selenium") {
        steps {
            sh "echo Lanzar los funcionales e2e sobre el frontend desplegado"
        }
    }
  }

  post {
    always {
      sh 'docker logout'
    }
  }
}