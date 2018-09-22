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



    }

}
