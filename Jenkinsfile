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
      // sh "git clone https://github.com/Homebrew/brew ~/.linuxbrew/Homebrew"
      // sh "mkdir ~/.linuxbrew/bin"
      // sh "ln -s ../Homebrew/bin/brew ~/.linuxbrew/bin"
      // sh "eval \$(~/.linuxbrew/bin/brew shellenv)"
      sh "curl https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh --output install.sh"
      sh "chmod 755 install.sh"
      sh "sudo install.sh"
      sh "rm install.sh"
      sh "brew install gitversion"
      sh "gitversion /output buildserver"
      sh "printenv"
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