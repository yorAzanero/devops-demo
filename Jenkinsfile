import hudson.model.* 
import hudson.EnvVars 
import groovy.json.JsonSlurperClassic 
import groovy.json.JsonBuilder 
import groovy.json.JsonOutput 
import java.net.URL
def DEPLOYMENT_OBJECT 
def DEPLOYMENT_ID 
@NonCPS 
def jsonParse(def json) { 
  new groovy.json.JsonSlurperClassic().parseText(json) 
}

pipeline {
  agent any
  environment {
      IMAGE_NAME = 'caleidos/yor-web'
      AWS_REGION = 'us-east-1'
      AWS_ACCOUNT = '033686261524'
      IMAGE_TAG = getShortCommitId()
      ENVIRONMENT = getEnvironment()
  }
  stages {
      stage ('Create Image') {
      steps {
        script {
          def imageTag = "${IMAGE_TAG}"
          def imageName = "${IMAGE_NAME}:${imageTag}"
          def repositoryName = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${imageName}"
          sh "\$(aws ecr get-login --no-include-email --region ${AWS_REGION})"
          sh "docker build -t ${imageName} ."
          sh "docker tag ${imageName} ${repositoryName}"
        }
      }
    }
  }
}

def getShortCommitId() {
    def gitCommit = env.GIT_COMMIT
    def shortGitCommit = "${gitCommit[0..6]}"
    return shortGitCommit
}

def getEnvironment(){
    if(isDevelop())
      return "dev"

    if(isRelease())
      return "qas"

    if(isMaster())
      return "prd"

    return "dev"
}

def isMaster() {
    return env.BRANCH_NAME == "master"
}

def isRelease() {
    return env.BRANCH_NAME ==~ '^release\\/[\\w\\d\\.]*$'
}

def isDevelop() {
    return env.BRANCH_NAME == "develop"
}