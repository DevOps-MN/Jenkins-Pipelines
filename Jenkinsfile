pipeline {
    environment {
    registry = "km20/pipeline"
    credentialsId = "docker_hub"
    dockerImage = ''
    }
  agent {
    kubernetes {
      label 'docker'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: jenkins
  containers:
  - name: docker
    image: docker:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
  volumes:
    - name: docker-sock
      hostPath:
        path: /var/run/docker.sock
    - name: m2
      persistentVolumeClaim:
        claimName: jenkins
"""
}
   }
  stages {
    stage("Checkout SCM"){
        steps {
            git credentialsId: 'git-creds', url: 'https://github.com/DevOps-MN/Jenkins.git'
        }
    }
    
    stage('Build') {
      steps {
        container('docker') {
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
        }
      }
    }
    stage('Deploy Image') {
      steps{
          container('docker') {
            script {
                docker.withRegistry('https://index.docker.io/v1/', 'docker_hub'){
                dockerImage.push()
                }
            }
          }
        }
    }
    stage ('changetag'){
      steps {
        sh "chmod +x changetag.sh"
        sh "./changetag.sh ${BUILD_NUMBER}"
     }
    }    
    stage('Deploy App') {
      steps {
        script {
          kubernetesDeploy(configs: "myweb-deploy.yaml", kubeconfigId: "mykubeconfig")
        }
      }
    }
  }
}
