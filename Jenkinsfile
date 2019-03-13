pipeline {
   environment {
    registry = "arkakundu1407/docker-pipeline"
    registryCredential = 'dockerhub'
    dockerImage = ''
    containerId = sh(script: 'docker ps -aqf "name=java-app"', returnStdout: true) //to store your container id , so that it can be deleted
   
  }
  agent any
    stages 
    {
        stage('Unit Test') {
      steps{
        script {
          sh 'python /var/jenkins_home/workspace/webapp-python-project/posts/tests.py'
        }
      }
    } 
       stage("Sonar scanner"){
          steps{
            sh 'rm -rf scanlatest.html'
            sh "/opt/sonar/bin/sonar-scanner \
  -Dsonar.projectKey=python-webapp \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://104.211.208.249:9000 \
  -Dsonar.login=65e9c19ce16f518f1c778442ae3f390ed76dd1ef"
         
          }
       }
     stage('Building image') {
      steps{
        script {
          //will pisck registry from variable defined
          //dockerImage = docker.build registry + ":$BUILD_NUMBER"
          dockerImage = docker.build registry
           //sh 'sed "s/latest/$BUILD_NUMBER/g" Application.yml > Application1.yml'
           //sh 'mv Application1.yml Application.yml'
        }
      }
    }

       
      stage('Push Image') {
      steps{
         script {
            docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
     
         stage('Docker Image Scanning') {
        steps{
           
           aquaMicroscanner imageName:'arkakundu1407/docker-pipeline:latest' , notCompliesCmd: 'exit 1', onDisallowed: 'fail', outputFormat: 'html'
           
        }
    } 
       
       
      /*stage('Cleanup') {
      when {
                not { environment ignoreCase: true, name: 'containerId', value: '' }
        }
      steps {
        sh 'docker stop ${containerId}'
        sh 'docker rm ${containerId}'
      }
    }*/
       
       
           
    stage('Server Hardening') {
      steps {
         
        sh 'git clone https://github.com/CISOfy/lynis'
         sh 'rm -rf /usr/local/lynis'
        sh 'mv lynis /usr/local/'
         sh 'chown -R root:root /usr/local/lynis'
         sh '/usr/local/lynis/lynis audit system > /var/jenkins_home/workspace/webapp-python-project/hardening-output.txt'
      }
    }
   /* stage('Remove Unused docker image') {
      steps{
        sh "docker rmi -f $registry:$BUILD_NUMBER"
      }
    }*/
stage ('Deploy application') {
       steps {
           kubernetesDeploy(
               kubeconfigId : 'kubeconfig',
               configs : 'Application.yml',
               enableConfigSubstitution : false
           )
       }
     }
              stage('ARACHNI Scanning') {
         steps {
            arachniScanner checks: '*', scope: [pageLimit: 3], url: 'http://13.71.114.235:80/posts/', userConfig: [filename: 'myConfiguration.json'], format: 'json'
         }
      }

 }
}
