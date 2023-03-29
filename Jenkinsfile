pipeline {

  agent {
    label 'docker'
  }


  environment {
    dockerimagename = "dockerid/nodeapp"
    dockerImage = ""
    HELM_PATH = "../deployment/charts"    
    RELEASE_NAME = "nodeapp"
    NAMESPACE = "postpayc2c"
  }


  stages {

    stage('Checkout Source') {
      steps {
        git 'https://github.com/<sachith>/nodeapp-jenkins'      }
    }

    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }

    stage('Test') {
      steps {
        sh "npm run test"
      }
    }

    stage('Pushing Image') {
      environment {
               registryCredential = 'dockerhublogin'
           }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("latest")
          }
        }
      }
    }

    stage('Deploying App to Test') {
       
    steps{
      sh "helm lint ${HELM_PATH}"
        kubeconfig(caCertificate: '', credentialsId: 'kubetest', serverUrl: '') {
          sh "helm upgrade --wait --set image.tag=${BUILD_NUMBER} -i -n ${NAMESPACE} ${RELEASE_NAME} ${HELM_PATH} -f ${HELM_PATH}/nonprodvalues.yaml    }
    }
    }
    }

    stage('Deploying App to Stage') {
  
    steps{
      when 
      { 
        branch 'release/*'      
      }

      
        kubeconfig(caCertificate: '', credentialsId: 'kubeprod', serverUrl: '') {
          sh "helm upgrade --wait --set image.tag=${BUILD_NUMBER} -i -n ${NAMESPACE} ${RELEASE_NAME} ${HELM_PATH} -f ${HELM_PATH}/values.yaml    }
    }
    }
    }

    stage('Deploying App to Prod') {
       
    steps{

      when 
      { 
        branch 'master'      
      }

        input 'Please Approve'

        kubeconfig(caCertificate: '', credentialsId: 'kubeprod', serverUrl: '') {
          sh "helm upgrade --wait --set image.tag=${BUILD_NUMBER} -i -n ${NAMESPACE} ${RELEASE_NAME} ${HELM_PATH} -f ${HELM_PATH}/values.yaml    }
    }
    }
    }

    post {
      always {
        echo 'done'
      }
    }
  }

