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
  }
}