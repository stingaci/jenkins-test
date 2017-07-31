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
  withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]){
    sh 'export AWS_DEFAULT_REGION=us-west-2; eval `aws ecr get-login | cut -d" " -f1,2,3,4,5,6,9`'
  }

  stage 'Test Image'
  // Read extra running options for image 
  try {
    def run_opts = readFile('run_opts') 
  } catch (err) {
    print ("Missing run_opts file. Running container with default running paramters") 
    run_opts = ""
  } 

  // Run Container
  try { 
    sh "docker run -d ${run_opts} ${app_name} > container.id"
    def container_id = readFile("container.id")
  catch (err) {
    error ("Failed to run container: ${err}")
  }

  // Verify Container didn't exit early
  sh "docker inspect -f {{.State.Running}} ${container_id} > running.status"
  def running_status = readFile ("running.status")
  if running_status != "false" {
    sh "docker inspect -f {{.State.ExitCode}} ${container_id} > running.exit_code"
    sh "docker inspect -f {{.State.Error}} ${container_id} > running.error"
    def exit_code = readFile("running.exit_code")
    def error = readFile("running.error")
    error ("Container exits upon creation with status code: ${exit_code} and error: ${error}")
  } 

  // Actually run the tests against running container
  try { 
    sh = "tests/main.sh"
  } catch (err) { 
    error ("Image tests failed with the following error(s): ${err}")
  }

  // Cleanup tests
  try {
    sh = "docker stop ${container_id}" 
  } catch (err) {
    print ("Docker stop command failed with the following error: ${err}")
  }

  stage 'Push Image'
  docker.withRegistry('https://400585646753.dkr.ecr.us-west-2.amazonaws.com') {
    docker.image("${app_name}").push("${app_version}_${app_revision}")
  }

}

