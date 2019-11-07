def asm = ASM.newObserver()
def buildInfo = Artifactory.newBuildInfo()
buildInfo.env.capture = true
def uploadSpec = """{
          "files": [
            {
              "pattern": "*.tar",
              "target":"generic-local/node-todo-frontend/${BUILD_NUMBER}/"
            }
          ]
        }"""

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
	 }
    }
    stage('cloneGit') {
      steps {
        git 'https://github.com/mattdugganibm/node-todo-frontend'
      }
    }
    stage('build npm') {
       steps {
	 sh('printenv | sort')
         sh 'npm install'
       }
    }
    stage('test') {
      steps {
        sh 'npm test'
      }
    }
    stage('build docker image') {
      steps{
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    stage('save docker image') {
      steps {
         script {
            docker.withRegistry( '', registryCredential ) {
                dockerImage.push("${env.BUILD_NUMBER}")
                dockerImage.push('latest')
             }
             save_rc = sh(returnStatus: true, script: "docker save -o $WORKSPACE/$dockerImageSaveFile $registry:$BUILD_NUMBER")
             echo "save_rc: $save_rc"
	     sh('pwd')
             sh('ls -altr')
          }
       }       
    }
    stage('deploy to artifactory'){
        buildInfo.setName "node-todo-frontend_artifactory"
        def server = Artifactory.server 'artifactory'
        server.upload spec: uploadSpec, buildInfo: buildInfo
        server.publishBuildInfo buildInfo
    }
    stage('cleanup') {
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
