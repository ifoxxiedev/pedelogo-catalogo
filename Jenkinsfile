pipeline {
  // Agent: Docker Images | KUbernetes | Any
  agent any

  // Stages from pipeline
  stages {

    // Stage
    stage ('Checkout Project Source') {

      // Steps for stage
      steps {
        git url: 'https://github.com/ifoxxiedev/pedelogo-catalogo.git', branch: 'main'
      }
    }

    // Stage
    stage ('Build Application') {
      steps {
        script {
          // Build Docker Image
          dockerapp = docker.build("obededoreto/api-produto:${env.BUILD_ID}", '-f ./src/PedeLogo.Catalogo.Api/Dockerfile .')
        }
      }
    }

    // Stage
    stage ('Push Image') {
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') { // Credentials Ref: dockerhub
            dockerapp.push('latest')
            dockerapp.push("${env.BUILD_ID}")
          }
        }
      }
    }

    // Stage
    stage ('Deploy kubernetes') {
      agent {
        kubernetes {
          cloud 'jadeerp-hml-cluster-k8s' // ref, cloud config
        }
      }
      environment {
        tag_version = "${env.BUILD_ID}"
      }

      steps {
        script {
          sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/api/deployment.yml'
          sh 'cat ./k8s/api/deployment.yml'
          
          kubernetesDeploy(configs: '**/k8s/**', kubeconfigId: 'kubernetes')
        }
      }
    }
  }
}