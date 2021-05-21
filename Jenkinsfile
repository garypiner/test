#!/usr/bin/env groovy
@Library("jenkins-library") _

def workerNodeId = jenkinsLib.getBuildName() + '-' + jenkinsLib.getBuildNumber()
def instance_type = "t3.large"

def release(files) {
  if (files instanceof List) {
    release_files = files
  } 
  else if (files.contains("*")) {
    release_files = findFiles glob: files
  } 
  else {
    release_files = files.split(",")
  }

  withCredentials([string(credentialsId: 'github-access-token', variable: 'GITHUB_TOKEN')]) {
    env.PATH="$PATH:/usr/bin:/usr/local/bin:/usr/local/go/bin:~/go/bin"
    try {
      go version

    }
    catch (e) {
      sh "sudo yum install golang -y"
    }
    sh "go get github.com/github-release/github-release"

    sh "github-release release --user garypiner --repo ${jenkinsLib.getBuildName()} --tag ${gitversion} --name \"${gitversion}\""
    for (file in release_files) {
      println file
      println file.split('/+')
      println file.split('/+').last()
      def split_file_name = file.split('/+').last()
      sh "github-release upload --user garypiner --repo ${jenkinsLib.getBuildName()} --tag ${gitversion} --name \"${split_file_name}\" --file ${file}"
    }
  }
}

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
      sh "zip -r test.zip test.py"
      sh "zip -r test2.zip test.py"
      sh "mkdir test && mv test.zip test/ && mv test2.zip test/"

      release("test/*.zip")
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