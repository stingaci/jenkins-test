#!/usr/bin/env groovy
node {
  
  stage 'Check Out'
  def scmVars = checkout scm

  stage 'Validate'
  // Verify Dockerfile exists 
  try { 
    sh '[ -f "Dockerfile" ]'
  } catch (err) {
    error ("Missing Dockerfile")
  }

  // Gather APP_NAME, APP_VERSION, APP_REVISION
  try {
    sh 'cat Dockerfile | grep APP_NAME | cut -d= -f2 |  tr -d "[:space:]" > app_name'
    sh 'cat Dockerfile | grep APP_VERSION | cut -d= -f2 |  tr -d "[:space:]" > app_version'
    sh 'cat Dockerfile | grep APP_REVISION | cut -d= -f2 |  tr -d "[:space:]" > app_revision'
  } catch (err) { 
    error ('Missing or malformed APP_NAME, APP_VERSION, APP_REVISION in Dockerfile')
  } 
  def app_name = readFile('app_name')
  def app_version = readFile('app_version')
  def app_revision = readFile('app_revision')

  stage 'Build Image'
  docker.build("${app_name}")

  stage 'Docker Auth'
  withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '<CREDENTIAL_ID>', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]){
    sh 'export AWS_DEFAULT_REGION=us-west-2; eval `aws ecr get-login`'
  }

  stage 'Push Image'
  docker.withRegistry('https://400585646753.dkr.ecr.us-west-2.amazonaws.com') {
    docker.image("${app_name}").push("${app_version}_${app_revision}")
  }
}

