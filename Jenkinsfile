def asm = ASM.newObserver()
def buildInfo = Artifactory.newBuildInfo()

pipeline {
  environment {
    registry = "mattduggan/node-todo-frontend"
    registryCredential = 'dockerhub'
    dockerImage = ''
  }
  agent any
  tools {nodejs "node" }
  stages {
      stage ('Artifactory configuration') {
          steps {
            rtServer (
                id: "artifactory",
                url: "http://artifactory:8081/artifactory",
		credentialsId: 'artifactory'
            )
            rtBuildInfo (
                buildName: currentBuild.fullProjectName,
                    captureEnv: true
            )
	  }
    }
    stage('Cloning Git') {
      steps {
        git 'https://github.com/mattdugganibm/node-todo-frontend'
      }
    }
    stage('Build') {
       steps {
         sh 'npm install'
       }
    }
    stage('Test') {
      steps {
        sh 'npm test'
      }
    }
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    stage('Deploy Image') {
      steps {
         script {
            docker.withRegistry( '', registryCredential ) {
                dockerImage.push("${env.BUILD_NUMBER}")
                dockerImage.push('latest')
             }
          }
          rtPublishBuildInfo (
              serverId: "artifactory"
          )
	  rtUpload (
              serverId: 'artifactory',
              spec: '''{
	          "files": [
                   {
                      "pattern": "*.json",
                      "target": "generic-local/node-todo-frontend"
                   }
                 ]
              }''', failNoOp: true
          )
       }       
    }
    stage('Remove Unused docker image') {
      steps{
	echo "Registry:Build = $registry:$BUILD_NUMBER"
        sh "docker rmi $registry:$BUILD_NUMBER"
	sh "docker rmi $registry:latest"
      }
    }
  }
  post {
        always {
            script {
              ASM.notifyASM asmObserver: asm, artModules: buildInfo.getModules()
            }
	    deleteDir()
        }
   }
}
