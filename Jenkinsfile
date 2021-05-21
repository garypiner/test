#!/usr/bin/env groovy
@Library("jenkins-library") _

def workerNodeId = jenkinsLib.getBuildName() + '-' + jenkinsLib.getBuildNumber()
def instance_type = "t3.large"

try {
  stage("worker") {
    workerLib.createEC2(workerNodeId, "awscentral", "prod", instance_type)
  }

  node(workerNodeId) {
    stage("Tag") {
      checkout scm: [$class: 'GitSCM', branches: [[name: 'refs/heads/'+env.BRANCH_NAME]], extensions: [[$class: 'CloneOption', noTags: false]], userRemoteConfigs: scm.userRemoteConfigs,]
      gitversion = sh(script: "docker run --rm -v \"\$(pwd):/repo\" gittools/gitversion:5.6.6 /repo | jq .SemVer  | sed 's/\"//g'" , returnStdout: true).trim()
      sh "printenv"
      echo gitversion
      withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]){    
        sh("git tag -a ${gitversion} -m \'jenkins\'")
        sh("git push https://${env.GIT_USERNAME}:${env.GIT_PASSWORD}@github.com/garypiner/${jenkinsLib.getBuildName()} --no-verify --tags")
      }
    }
    stage("release") {
      withCredentials([string(credentialsId: 'github-access-token', variable: 'GITHUB_TOKEN')]) {
        sh "zip -r artifacts.zip test.py"
        withEnv(['GITHUB_API=https://api.github.com/api/v3']) {
          env.PATH="$PATH:/usr/bin:/usr/local/bin:/usr/local/go/bin:~/go/bin"
          try {
            go version
            
          }
          catch (e) {
            sh "sudo yum install golang -y"
          }
          sh "go get github.com/github-release/github-release"
          sh "github-release delete --user garypiner --repo ${jenkinsLib.getBuildName()} --tag ${gitversion}"

          sh "github-release release --user garypiner --repo ${jenkinsLib.getBuildName()} --tag ${gitversion} --name \"${gitversion}\""

          sh "github-release upload --user garypiner --repo ${jenkinsLib.getBuildName()} --tag ${gitversion} --name \"${jenkinsLib.getBuildName()}-${gitversion}.zip\" --file artifacts.zip"
        }
      }
    }
  }
}

catch (e) {
    currentBuild.result = "FAILED"
    throw e
}
finally {
    node("master") {
      cleanWs()
      workerLib.destroyEC2(workerNodeId, "awscentral", "prod", instance_type)
    }
}