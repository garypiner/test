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
      checkout scm
      gitversion = sh(script: "docker run --rm -v \"\$(pwd):/repo\" gittools/gitversion:5.6.6 /repo | jq .SemVer  | sed 's/\"//g'" , returnStdout: true).trim()
      sh "printenv"
      echo gitversion
      gitLib.tag(gitversion)
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