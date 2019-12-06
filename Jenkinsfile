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
  stages {
    stage("Building Artifact") { 
      steps { 
        sh ''' 
        #!/bin/bash -xe 
        cd $WORKSPACE 
        zip -r $BUILD_TAG.zip * 
        ''' 
      } 
    }
    stage("Uploading S3 Artifact") { 
      steps { 
        sh ''' 
        #!/bin/bash -xe 
        aws s3 mv $BUILD_TAG.zip s3://ci-workshop-devops/yor/Artifact/ --region us-east-1 
        ''' 
      } 
    }
    stage("Deploy") {
      steps {
        script {
          DEPLOYMENT_ID = sh (returnStdout: true, script: 'aws deploy create-deployment --application-name yor --deployment-group-name DEV --s3-location bucket=ci-workshop-devops,key=yor/Artifact/$BUILD_TAG.zip,bundleType=zip --file-exists-behavior OVERWRITE --region us-east-1').trim()     
          DEPLOYMENT_OBJECT = jsonParse(DEPLOYMENT_ID)
          echo "Deployment-object is => ${DEPLOYMENT_ID}"
          echo "Deployment-Id is => ${DEPLOYMENT_OBJECT.deploymentId}"
        }
        timeout(time: 5, unit: 'MINUTES'){                                         
            awaitDeploymentCompletion("${DEPLOYMENT_OBJECT.deploymentId}")
        }
      }
    }
  }
}