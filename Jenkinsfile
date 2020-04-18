#!groovy

node('master') {

    stage('Clean Workspace') {
        deleteDir()
    }

    stage('Checkout Tools-Belt') {
	     checkout scm
    }

    stage('build') {

      def branchName = env.BRANCH_NAME
      def branch = branchName.tokenize('/')
      def artifactId = "karthik-app"

      withCredentials([usernamePassword(credentialsId: 'docker-admin', usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {
          sh 'docker login kardocker.azurecr.io -u $dockerUser -p $dockerPassword'

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
          sh "docker build -t kardocker.azurecr.io/${artifactId}:${version} -f Dockerfile ."
          sh "docker push kardocker.azurecr.io/${artifactId}:${version}"
          sh 'docker logout'

      }
    }
}
