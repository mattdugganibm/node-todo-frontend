def asm = ASM.newObserver()
def buildInfo = Artifactory.newBuildInfo()

pipeline {
  environment {
    registry = "mattduggan/node-todo-frontend"
    dockerImageSaveFile = "node-todo-frontend.tar" 
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
	 sh('printenv | sort')
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
             save_rc = sh(returnStatus: true, script: "docker save -o $WORKSPACE/$dockerImageSaveFile $registry:$BUILD_NUMBER")
             echo "save_rc: $save_rc"
          }
          rtPublishBuildInfo (
              serverId: "artifactory"
          )
	  rtUpload (
              serverId: 'artifactory',
              spec: '''{
	          "files": [
                   {
                      "pattern": "*.tar"
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
	   // deleteDir()
        }
   }
}
