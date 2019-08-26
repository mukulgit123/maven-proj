#!groovy
pipeline{
agent { label 'master' }
environment{
registry = "mukuldoc/mvnjenkins"
registryUrl = 'https://registry.hub.docker.com'
registryName = 'mukuldoc/mvnjenkins'
registryCredential = 'docker_registry_server'
def img=''
extravar=''
SLACK_CHANNEL_NAME='#sonarqube'
}
stages{
    stage('Checkout'){
        steps{
       checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CleanCheckout']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '5ed90783-85d8-44fc-b3e2-5581a7e848a4', url: 'https://github.com/mukulgit123/maven-proj.git']]])
   
        }
        }
  stage('Build & Package') {
    withSonarQubeEnv('Sonarqube') {
        sh 'mvn -f ./pom.xml clean test sonar:sonar'
    }
} 
  stage('Build') { steps{
       echo '"Building Maven1"'
       sh """
       sudo mvn clean package
       """
   }
   }
   
   stage('Copy Artifact to Docker'){
       steps{
           sh """
           pwd
           echo "Copying Artifact to Docker Folder."
           cp ./target/*.jar ./Docker/
           """
       }
   }
       stage('Build Docker Image') {
      steps{
         script {
                img = docker.build("mukuldoc/mvnjenkins:${env.BUILD_ID}", "-f Docker/Dockerfile './Docker/'")
                println "Newly Generated Image, " + img.id
                
            }
        }
      }
stage('Deploy Image') {
      steps{
         script {
            docker.withRegistry("${env.registryUrl}", 'docker_registry_server') {
                img.push()
            }
        }
      }
}
   stage('Remove Image'){
       steps{
           sh """
           extravar=`docker images | grep mvnjenkins |awk -F" " '{print \$3}' | head -1| xargs`
           docker rmi -f \$extravar
           """
       }
   }
}
post {
        success {
            slackSend (channel: "${SLACK_CHANNEL_NAME}", color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' Application Deployed Successfully/${env.BUILD_URL})")
        }
        failure {
            slackSend (channel: "${SLACK_CHANNEL_NAME}", color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' Application Deployment Failed (${env.BUILD_URL})")
        }
    }
}
