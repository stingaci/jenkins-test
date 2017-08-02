Image-Building Piepline 
=======================

This jenkins file can be used to create a Jenkins pipeline that can be used to 
build Docker images and push them into an AWS image registry. 

In addition to the Jenkinsfile, this pipeline will require the following items:

* A secret credential with name aws_creds of kind: "Username with password" where the USERNAME is the value of AWS_ACCESS_KEY_ID, and the PASSWORD is the value of AWS_SECRET_ACCESS_KEY

* A Jenkins parameter called AWS_REGION which specifies which AWS region this image will be pushed to

* A Jenkins parameter called AWS_ACCOUNT_ID which specifies which account number the registry belongs to

* A Jenkins parameter called IMAGE_REPO_URL which specifies which repository Jenkins should pull containing a Dockerfile 

* A Jenkins parameter called IMAGE_REPO_BRANCH which specifies which branch of the image repository should be built 

This pipeline has five stages:

1. "Check Out" Stage -- Pulls the branch specified by IMAGE_REPO_BRANCH of the repository specified by IMAGE_REPO_URL

2. "Validate" Stage -- Validates that all necessary information is available before continuing to build the image (see Docker image template repo README)

3. "Build Image" Stage -- Uses the docker tools to build the image

4. "Test Image" Stage -- Uses the tests defined by the owner of the repo in order to determine wether or not this image is behaving as it should 

5. "Push Image" stage -- Uses the aws_creds with the AWS_REGION, and AWS_ACCOUNT_ID in order to push this image to the AWS repository, if and only if the IMAGE_REPO_BRANCH=master

In order to use this pipeline you must have Docker installed on the Jenkins node who will be executing this pipeline.
