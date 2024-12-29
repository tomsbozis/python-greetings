pipeline {
    agent any

    triggers {
        pollSCM('*/1 * * * *')
    }

    parameters {
        string(name: 'DOCKER_USER', defaultValue: '', description: 'Docker Hub Username')
        password(name: 'DOCKER_PASSWORD', defaultValue: '', description: 'Docker Hub Password')
    }

    stages {
        stage('build-docker-image') {
            steps{
                getGitRepository()
                buildDockerImage()
            }
        }
        stage('deploy-dev') {
            steps {
                deploy("dev")
            }
        }
        stage('tests-on-dev') {
            steps {
                runTest("dev")
            }
        }
        stage('deploy-to-stg') {
            steps {
                deploy("stg")
            }
        }
        stage('tests-on-stg') {
            steps {
                runTest("stg")
            }
        }
        stage('deploy-to-prd') {
            steps {
                deploy("prod")
            }
        }
        stage('tests-on-prd') {
            steps {
                runTest("prod")
            }
        }
    }
}

def getGitRepository(){
    echo "Getting Git repository via script"
    git branch: 'main', url: 'https://github.com/tomsbozis/python-greetings'
}

def buildDockerImage() {
    echo "Build docker image "
    sh "docker build --no-cache -t tomsbozis/python-greetings-app:latest ."

    echo "Log in to Docker Hub for push step"
    sh "echo ${params.DOCKER_PASSWORD} | docker login -u ${params.DOCKER_USER} --password-stdin"

    echo "Pushing docker image to the docker registry"
    sh "docker push tomsbozis/python-greetings-app:latest"
}


def deploy(String environment){
    echo "Deployment triggered to ${environment} environemnt "
    sh "docker pull tomsbozis/python-greetings-app:latest"

    String lowercaseEnv = environment.toLowerCase()
    sh "docker compose stop greetings-app-${lowercaseEnv}"
    sh "docker compose rm greetings-app-${lowercaseEnv}"
    sh "docker compose up -d greetings-app-${lowercaseEnv}"
}

def runTest(String environment){
   echo "Tests triggered on ${environment} environemnt "
   sh "docker pull tomsbozis/api-tests-final:latest"
   sh "docker run --network=host --rm tomsbozis/api-tests-final run greetings greetings_${environment}"
}