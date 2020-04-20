#!groovy

node('master') {

    stage('Clean Workspace') {
        deleteDir()
    }

    stage('Checkout Tools-Belt') {
	     checkout scm
    }

    stage('build') {
      dir('code') {
          def branchName = env.BRANCH_NAME
          def branch = branchName.tokenize('/')
          def artifactId = "karthik-app"

          withCredentials([usernamePassword(credentialsId: 'docker-admin', usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {
              sh 'sudo docker login kardocker.azurecr.io -u $dockerUser -p $dockerPassword'

              if ((branchName.startsWith("rc/") || branchName.startsWith("hf/")) && branch.size() >= 2) {
                  def release = branch[1]
                  if (release =~ /^(\d+\.)?(\d+\.)?(\*|\d+[a-z]?)$/) {
                      version = release
                  }
              } else if (branchName == 'development') {
                  version = 'latest'
              } else {
                  sh "echo 'Build & push will be executed only for development, rc/*, hf/* branches'"
              }
              sh "npm install --silent && npm install @vue/cli@3.7.0 -g"
              sh "npm run build"
              sh "sudo cp ./../config/vue.config.js ."
              sh "cd dist && find dist -type f "
              sh "cd .."
              sh "sudo docker build -t kardocker.azurecr.io/${artifactId}:${version} -f ProdDockerfile ."
              sh "sudo docker push kardocker.azurecr.io/${artifactId}:${version}"
              sh "sudo docker logout"

          }
      }
    }
}
