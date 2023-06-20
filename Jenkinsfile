pipeline {
  agent {
    label 'worker'
  }
  stages {
    stage('Git Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      parallel {
        stage('Build Docker Image') {
          steps {
            sh 'cd vote'
            sh 'sudo aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 857269734878.dkr.ecr.us-east-1.amazonaws.com'
            sh 'sudo docker build -t 857269734878.dkr.ecr.us-east-1.amazonaws.com/jenkinsfile-cicd:${BUILD_NUMBER} .'
            sh 'sudo docker push 857269734878.dkr.ecr.us-east-1.amazonaws.com/jenkinsfile-cicd:${BUILD_NUMBER}'
          }
        }

        stage('Unit Testing') {
          steps {
            sh 'echo Run the Test Cases'
          }
        }

      }
    }

    stage('Deploy in ECS') {
      steps {
        script {
          sh'''
ECR_IMAGE="857269734878.dkr.ecr.us-east-1.amazonaws.com/jenkinsfile-cicd:${BUILD_NUMBER}"
TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition jenkinsfile-cicd --region "us-east-1")
NEW_TASK_DEFINTIION=$(echo $TASK_DEFINITION | jq --arg IMAGE "$ECR_IMAGE" '.taskDefinition | .containerDefinitions[0].image = $IMAGE | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities) | del(.registeredAt) | del(.registeredBy)')
NEW_TASK_INFO=$(aws ecs register-task-definition --region "us-east-1" --cli-input-json "$NEW_TASK_DEFINTIION")
NEW_REVISION=$(echo $NEW_TASK_INFO | jq '.taskDefinition.revision')
aws ecs update-service --cluster vote-app \
--service vote \
--region us-east-1 \
--task-definition ${TASK_FAMILY}:${NEW_REVISION}'''
        }

      }
    }

    stage('Adding New Stage') {
      steps {
        sh 'echo adding a new stage'
      }
    }

  }
  environment {
    AWS_DEFAULT_REGION = 'us-east-1'
    SERVICE_NAME = 'vote'
    TASK_FAMILY = 'jenkinsfile-cicd'
    ECS_CLUSTER = 'vote-app'
  }
  post {
    always {
      deleteDir()
      sh 'sudo docker rmi 857269734878.dkr.ecr.us-east-1.amazonaws.com/jenkinsfile-cicd:${BUILD_NUMBER}'
    }

  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    disableConcurrentBuilds()
    timeout(time: 1, unit: 'HOURS')
  }
}
