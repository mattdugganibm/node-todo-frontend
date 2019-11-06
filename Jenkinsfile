def asm = ASM.newObserver()

def buildContext = createBuildContext([
  artRegion: 'na',
  artPullRegion: 'na',
  genericRepository: 'generic-local'
])

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
                dockerImage.push()
             }
          }
          rtPublishBuildInfo (
              serverId: "artifactory"
          )
       }       
    }
    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }
  }
  post {
        always {
            script {
              ASM.notifyASM asmObserver: asm, artModules: buildContext.artBuildInfo.getModules()
            }
        }
   }
}
