pipeline{

  agent {
      kubernetes {
        yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: lhamaoka/nodo-nodejs-practica-final:1.0
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-socket-volume
    securityContext:
      privileged: true
  - name: nodo-java
    image: lhamaoka/nodo-java-practica-final:1.0
    command:
    - cat
    imagePullPolicy: IfNotPresent
    tty: true
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

  environment {
    registryCredential='dockerhub_credentials'
    registryFrontend = 'lhamaoka/practica-final-frontend'
  }

  stages {
    // stage("1.- Prepare environment") {
    //     steps {
    //         sh "node -v"
    //     }
    // }

    // stage("2.- Build") {
    //     steps {
    //         sh "echo Consistirá en construir la aplicación Angular"
    //         sh 'npm install && npm run build'
    //     }
    // }

    // stage("3.- Quality Tests") {
    //     steps {
    //         sh "echo Comprobación de la calidad del código con Sonarqube."
    //         sh 'SonarQube analysis'
    //         withSonarQubeEnv(credentialsId: "sonarqube-credentials", installationName: "sonarqube-server"){
    //           sh 'npm run sonar'
    //         }
    //         sh 'Quality Gate'
    //         timeout(time: 10, unit: "MINUTES") {
    //           script {
    //             def qg = waitForQualityGate()
    //             if (qg.status != 'OK') {
    //               error "Pipeline aborted due to quality gate failure: ${qg.status}"
    //             }
    //           }
    //         }
    //     }
    // }

    // stage("4.- Build & Push") {
    //   steps {
    //     sh "echo Construcción de la imagen con Kaniko y subida de la misma a vuestro repositorio personal en Docker Hub"
    //     script {
    //       dockerImage = docker.build registryFrontend + ":$BUILD_NUMBER"
    //       docker.withRegistry( '', registryCredential) {
    //         dockerImage.push()
    //       }

    //       dockerImage = docker.build registryFrontend + ":latest"
    //       docker.withRegistry( '', registryCredential) {
    //         dockerImage.push()
    //       }
    //     }
    //   }
    // }

    stage("5.- Run test environment") {
      steps {
        sh "echo Iniciar un pod o contenedor con la imagen que acabamos de generar"
        script {
          if(fileExists("launcher")){
            sh 'rm -r launcher'
          }
        }
        sh "git clone https://github.com/lhamaoka/manifest_launcher.git launcher"
        sh "kubectl apply -f launcher/deploys/frontend/manifest.yaml --kubeconfig=launcher/config/config"
      }
    }

    stage("6.- Selenium") {
        steps {
          container('nodo-java'){
            sh "echo Lanzar los funcionales e2e sobre el frontend desplegado"
            sh 'echo Run function testing E2E'
            sh 'mvn clean verify -Dwebdriver.remote.url=https://daita-selenium.loca.lt/wd/hub -Dwebdriver.remote.driver=chrome -Dchrome.switches="--no-sandbox,--ignore-certificate-errors,--homepage=about:blank,--no-first-run,--headless"'

            sh 'echo Generate Cucumber Report'
            sh 'mvn serenity:aggregate'

            publishHTML(target: [
                reportName : 'Serenity',
                reportDir:   'target/site/serenity',
                reportFiles: 'index.html',
                keepAll:     true,
                alwaysLinkToLastBuild: true,
                allowMissing: false
            ])
          }
        }
    }
  }

  post {
    always {
      sh 'docker logout'
    }
  }
}