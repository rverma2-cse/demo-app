def CONTAINER_NAME="jenkins-pipeline-test"
def CONTAINER_TAG="latest"
def DOCKER_HUB_USER="rverma2"
def HTTP_PORT="8090"

node {

    stage('Initialize'){
        def dockerHome = tool 'Docker'
        def mavenHome  = tool 'Maven'
        env.PATH = "${dockerHome}/bin:${mavenHome}/bin:${env.PATH}"
        echo "Application path: ${env.PATH} "
    }

    stage('Checkout') {
         checkout([$class: 'GitSCM',
             branches: [[name: '*/main']],
             doGenerateSubmoduleConfigurations: false,
             extensions: [[$class: 'CleanCheckout']],
             submoduleCfg: [],
             userRemoteConfigs: [[credentialsId: 'GitCreds', url: 'https://github.com/rverma2-cse/demo-app.git']]
         ])
    }

    stage('Build'){
        sh "mvn clean install"
    }

    stage("Image Prune"){
        imagePrune(CONTAINER_NAME)
    }

    stage('Image Build'){
        imageBuild(CONTAINER_NAME, CONTAINER_TAG)
    }

    stage('Push to Docker Registry'){
        withCredentials([usernamePassword(credentialsId: 'DockerCreds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            pushToImage(CONTAINER_NAME, CONTAINER_TAG, USERNAME, PASSWORD)
        }
    }

    stage('Run App'){
        runApp(CONTAINER_NAME, CONTAINER_TAG, DOCKER_HUB_USER, HTTP_PORT)
    }

}

def imagePrune(containerName){
    try {
        sh "docker image prune -f"
        sh "docker stop $containerName"
    } catch(error){}
}

def imageBuild(containerName, tag){
    sh "docker build -t $containerName:$tag  -t $containerName --pull --no-cache ."
    echo "Image build complete"
}

def pushToImage(containerName, tag, dockerUser, dockerPassword){
    sh "docker login -u $dockerUser -p $dockerPassword"
    sh "docker tag $containerName:$tag $dockerUser/$containerName:$tag"
    sh "docker push $dockerUser/$containerName:$tag"
    echo "Image push complete"
}

def runApp(containerName, tag, dockerHubUser, httpPort){
    sh "docker pull $dockerHubUser/$containerName"
    sh "docker run -d --rm -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
    echo "Application started on port: ${httpPort} (http)"
}