#!/usr/bin/env groovy
node {
  
  stage 'Check Out'
  checkout([$class: 'GitSCM', branches: [[name: '*/${BRANCH_NAME}']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/stingaci/jenkins-test']]])

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
    sh "docker run -d ${run_opts} ${app_name} > container_id"
  } catch (err) {
    error ("Failed to run container: ${err}")
  }

  // Verify Container didn't exit early
  def container_id = readFile("container_id")
  sh "docker inspect -f {{.State.Running}} ${container_id} > running.status"
  def running_status = readFile ("running.status")
  if ( running_status == "false" ) {
    sh "docker inspect -f {{.State.ExitCode}} ${container_id} > running.exit_code"
    sh "docker inspect -f {{.State.Error}} ${container_id} > running.error"
    def exit_code = readFile("running.exit_code")
    def error_msg = readFile("running.error")
    error ("Container exits upon creation with status code: ${exit_code} and error: ${error_msg}")
  } 

  // Actually run the tests against running container
  try { 
    sh "chmod 755 tests/main.sh && tests/main.sh"
  } catch (err) { 
    error ("Image tests failed with the following error(s): ${err}")
  }

  // Cleanup tests
  try {
    sh "docker stop ${container_id}" 
  } catch (err) {
    error ("Docker stop command failed with the following error: ${err}")
  }

  // If master push image
  if ("${BRANCH_NAME}" == "master") {
    stage 'Push Image'
    try {
      docker.withRegistry('https://400585646753.dkr.ecr.us-west-2.amazonaws.com') {
        docker.image("${app_name}").push("${app_version}_${app_revision}")
      }
    } catch (err) {
      withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]){
        sh 'export AWS_DEFAULT_REGION=${AWS_REGION}; eval `aws ecr get-login | cut -d" " -f1,2,3,4,5,6,9`'
      }
      docker.withRegistry('https://400585646753.dkr.ecr.us-west-2.amazonaws.com') {
        docker.image("${app_name}").push("${app_version}_${app_revision}")
      }
   }
  }

}

