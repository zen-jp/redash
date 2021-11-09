def ECR_REPO_NAME = "redash"
def AWS_REGION = ["ap-northeast-1"]
def BRANCH_NAME

pipeline {

    agent {
        node {
            label 'slave'
        }
    }

    options {
        ansiColor('xterm')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        DOCKER_PATH = tool (name: "docker-latest", type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool')
        AWS_ACCOUNT = sh (script: "aws sts get-caller-identity | jq -r '.Account'", returnStdout: true).trim()
        BRANCH_NAME = env.BRANCH_NAME
    }

    stages {
        stage('Docker build'){
            steps {
                 script {
                     sh "docker build -t ${ECR_REPO_NAME}:${BRANCH_NAME} -f .Dockerfile . --no-cache"
                 }
            }
        }

        stage("Push image"){
            steps {
                script {
                    try {
                        repo_status = sh(script: "aws ecr describe-repositories --repository-names ${ECR_REPO_NAME} --region ${AWS_REGION}", returnStatus: true)
                        if(repo_status != 0){
                            sh "aws ecr create-repository --repository-name ${ECR_REPO_NAME} --region ${AWS_REGION}"
                        }
                        sh "eval \$(aws ecr get-login --region ${AWS_REGION} --no-include-email)"
                        sh "docker tag ${ECR_REPO_NAME}:${BRANCH_NAME} ${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${BRANCH_NAME}"
                        sh "docker push ${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${BRANCH_NAME}"
                        sh "docker tag ${ECR_REPO_NAME}:${BRANCH_NAME} ${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:latest"
                        sh "docker push ${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:latest"
                        sh "docker rmi ${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${BRANCH_NAME}"
                        currentBuild.result = "SUCCESS"
                    } finally {
                        sh "docker rmi ${ECR_REPO_NAME}:${BRANCH_NAME}"
                    }
                }
            }
        }
    }
}
