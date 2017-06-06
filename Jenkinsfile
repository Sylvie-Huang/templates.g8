#!/usr/bin/groovy

node {
  stage('Build') {
    checkout scm
    sh 'ls -l'
    sbt = docker.image('hokiegeek/scala-sbt:0.13.12-2.11.8')
    sbt.inside("-v $PWD:/app") {
      sh 'sbt clean assembly'
    }
    archiveArtifacts artifacts: 'target/scala-2.11/*.jar', fingerprint: true
  }

  stage('Upload') {
    if (env.BRANCH_NAME == "master") {
      withAWS(credentials: 'iam:dev:cde.jenkins') {
        s3Upload(file:'target/scala-2.11',
            bucket:'wwi-app-core-data-etl-devstage3', 
            path:"apps/${env.JOB_NAME}/${env.BUILD_NUMBER}")
      }
    } else {
      echo "Not upload because ${env.BRANCH_NAME} is not master"
    }
  }

  stage('Report') {
    if (currentBuild.currentResult != 'SUCCESS') {
      slackSend color: 'bad', message: "Build ${env.BUILD_NUMBER} failed for ${env.JOB_NAME} -- ${currentBuild.absoluteUrl}"

    } else if (currentBuild.previousBuild != null && currentBuild.previousBuild.currentResult != 'SUCCESS') {
      slackSend color: 'good', message: "${env.JOB_NAME} back to normal"

    } else {
      echo 'Nothing new to report'
    }
  }
}

