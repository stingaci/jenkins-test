#!/usr/bin/env groovy
node {
  
  stage 'Check Out'
  checkout([$class: 'GitSCM', branches: [[name: '*/${IMAGE_REPO_BRANCH}']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: '${IMAGE_REPO_URL}']]])

  stage 'Validate'
  // Verify Dockerfile exists 
  if (!fileExists('Dockerfile')){
    error ("Missing Dockerfile")
  }

  // Gather APP_NAME, APP_VERSION, APP_REVISION
  def app_name = sh (script: 'cat Dockerfile | grep APP_NAME | cut -d= -f2 |  tr -d "[:space:]"', returnStdout: true)
  def app_version = sh (script: 'cat Dockerfile | grep APP_VERSION | cut -d= -f2 |  tr -d "[:space:]"', returnStdout: true)
  def app_revision = sh (script: 'cat Dockerfile | grep APP_REVISION | cut -d= -f2 |  tr -d "[:space:]"', returnStdout: true)
  if ( !app_name || !app_version  || !app_revision) {
    error ('Missing or malformed APP_NAME, APP_VERSION, APP_REVISION in Dockerfile')
  } 

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
  def container_id = sh (script: "docker run -d ${run_opts} ${app_name}", returnStdout: true)

  // Verify Container didn't exit early
  def running_status = sh (script: "docker inspect -f {{.State.Running}} ${container_id}", returnStdout: true)
  if ( running_status == "false" ) {
    def exit_code = sh (script: "docker inspect -f {{.State.ExitCode}} ${container_id}", returnStdout: true)
    def error_msg = sh (script: "docker inspect -f {{.State.Error}} ${container_id}", returnStdout: true) 
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
  if ("${IMAGE_REPO_BRANCH}" == "master") {
    stage 'Push Image'
    try {
      withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]){
       sh "export AWS_DEFAULT_REGION=${AWS_REGION}; if [ `aws ecr describe-repositories | grep repositoryName | grep \"${app_name}\" | wc -l` -eq 0 ]; then aws ecr create-repository --repository-name ${app_name}; fi"
      }
      docker.withRegistry("https://${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com") {
        docker.image("${app_name}").push("${app_version}_${app_revision}")
      }
    } catch (err) {
      withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]){
       sh 'export AWS_DEFAULT_REGION=${AWS_REGION}; eval `aws ecr get-login | cut -d" " -f1,2,3,4,5,6,9`'
       sh "export AWS_DEFAULT_REGION=${AWS_REGION}; if [ `aws ecr describe-repositories | grep repositoryName | grep \"${app_name}\" | wc -l` -eq 0 ]; then aws ecr create-repository --repository-name ${app_name}; fi"
      }
      docker.withRegistry("https://${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com") {
        docker.image("${app_name}").push("${app_version}_${app_revision}")
      }
   }
  }

}

