#!/usr/bin/env groovy
node {
  stage 'Docker build'
  def scmVars = checkout scm
  docker.build('testimage')
 
  stage 'Docker push'
  docker.withRegistry('400585646753.dkr.ecr.us-west-2.amazonaws.com/test', 'ecr:us-east-2:9c393ae1-fc62-47f3-abb9-0783d0aa9fd9  ') {
    docker.image('testimage').push('latest')
  }
}

