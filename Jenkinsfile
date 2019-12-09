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
    stage ('Push Image') {
      steps {
        script {
          def imageTag = "${IMAGE_TAG}"
          def imageName = "${IMAGE_NAME}:${imageTag}"
          def repositoryName = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${imageName}"
          sh "docker push ${repositoryName}"
        }
      }
    }
    // stage ('Deploy') {
    //   steps {
    //     script {
    //       def imageTag = "${IMAGE_TAG}"
    //       def imageName = "${IMAGE_NAME}:${imageTag}"
    //       def containerName = "yor"
    //       def containerPort = 80
    //       def applicationName = "AppECS-Yorweb-Yorweb"
    //       def deploymentGroupName = "DgpECS-Yorweb-Yorweb"
    //       def taskDefinitionName = "yor-web"

    //       sh  "                                                                     \
    //       sed -e 's;%REPO%;${imageName};g'\
    //           -e 's;%ENVIRONMENT%;${ENVIRONMENT};g'\
    //           -e 's;%ENVIRONMENTU%;${ENVIRONMENT.toUpperCase()};g'\
    //           -e 's;%CONTAINERNAME%;${containerName};g'\
    //           -e 's;%CONTAINERPORT%;${containerPort};g'\
    //           -e 's;%TASKDEFINITIONNAME%;${taskDefinitionName};g'\
    //               aws/task-definition.json >\
    //               aws/task-definition-${imageTag}.json\
    //       "
    //       TASK_DEFINITION = sh (returnStdout: true, script:"                                                                     \
    //       aws ecs register-task-definition --region us-east-1 --family yor-web                \
    //                                           --cli-input-json file://aws/task-definition-${imageTag}.json        \
    //       ")
    //       TASK_DEFINITION_OBJECT = jsonParse(TASK_DEFINITION)

    //       def content = "version: 0.0 \
    //       \nResources: \
    //       \n  - TargetService: \
    //       \n      Type: AWS::ECS::Service \
    //       \n      Properties: \
    //       \n        TaskDefinition: \"${TASK_DEFINITION_OBJECT.taskDefinition.taskDefinitionArn}\" \
    //       \n        LoadBalancerInfo: \
    //       \n          ContainerName: \"${containerName}\" \
    //       \n          ContainerPort: ${containerPort}"

    //       DEPLOYMENT_ID = sh (returnStdout: true, script: "aws deploy create-deployment --application-name ${applicationName} --deployment-group-name ${deploymentGroupName} --revision \"revisionType='String',string={content='${content}'\"}  --region ${AWS_REGION}").trim()
    //       DEPLOYMENT_OBJECT = jsonParse(DEPLOYMENT_ID)
    //       echo "Deployment-object is => ${DEPLOYMENT_ID}"
    //       echo "Deployment-Id is => ${DEPLOYMENT_OBJECT.deploymentId}"
    //     }
    //     timeout(time: 10, unit: 'MINUTES'){                               
    //         awaitDeploymentCompletion("${DEPLOYMENT_OBJECT.deploymentId}")                            
    //     }     
    //   }
    // }
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