def project = 'beemob-test'
def  appName = 'say-my-name'
def  feSvcName = "${appName}-frontend"
def  imageTag = "gcr.io/${project}/${appName}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
def mvnHome = tool 'M3'
def pom = readMavenPom file: 'pom.xml'
def appVersion = pom.version
//def imageTag = "beemob-test/say-my-name:${appVersion}"

pipeline {
  agent {
    kubernetes {
      label 'say-my-name'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: cd-jenkins
  containers:
  - name: java
    image: openjdk:8u111-jdk-alpine
    command:
    - cat
    tty: true
  - name: gcloud
    image: gcr.io/cloud-builders/gcloud
    command:
    - cat
    tty: true
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
"""
}
  }

  stages {
    //stage('Checkout'){
     // steps {
     //   checkout scm
     // }
    //}

    stage('Build the JAR') {
      steps {
        container('java') {
          sh "${mvnHome}/bin/mvn -Dmaven.test.failure.ignore clean package"
          //step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
        }
      }
    }

    stage('Build and push image with Container Builder') {
      steps {
        container('gcloud') {
          sh "PYTHONUNBUFFERED=1 gcloud container builds submit -t ${imageTag} ."
        }
      }
    }

    stage('Deploy Staging') {
      // staing branch
      when { branch 'staging' }
      steps {
        container('kubectl') {
         // Create namespace if it doesn't exist
            sh("kubectl get ns staging || kubectl create ns staging")

          // Change deployed image in canary to the one we just built
            sh("sed -i.bak 's#gcr.io/cloud-solutions-images/say-my-name:latest#${imageTag}#' ./k8s/deployments/staging/*.yaml")
            sh("kubectl --namespace=staging apply -f k8s/services/staging")
            sh("kubectl --namespace=staging apply -f k8s/deployments/staging")
            sh("echo http://`kubectl --namespace=staging get service/${feSvcName} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${feSvcName}")
        }
      }
    }

    stage('Deploy Production') {
      // Production branch
      when { branch 'master' }
      steps{
        container('kubectl') {
        sh("kubectl get ns production || kubectl create ns production")

        // Change deployed image in canary to the one we just built
          sh("sed -i.bak 's#gcr.io/cloud-solutions-images/say-my-name:latest#${imageTag}#' ./k8s/deployments/production/*.yaml")
          sh("kubectl --namespace=production apply -f k8s/services/production")
          sh("kubectl --namespace=production apply -f k8s/deployments/production")
          sh("echo http://`kubectl --namespace=production get service/${feSvcName} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${feSvcName}")
        }
      }
    }

  }
}
