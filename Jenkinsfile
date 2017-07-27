#!/usr/bin/env groovy
pipeline {
    agent any 

    

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("testimage")
    }

    stage('Push image') {
        docker.withRegistry('400585646753.dkr.ecr.us-west-2.amazonaws.com/test', 'ecr:us-east-2:9c393ae1-fc62-47f3-abb9-0783d0aa9fd9  ') {
          docker.image('testimage').push('latest')
        }
    }
}

