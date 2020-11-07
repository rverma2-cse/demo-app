def CONTAINER_NAME="jenkins-pipeline-test"
def CONTAINER_TAG="v1.1"
def DOCKER_HUB_USER="rverma2"
def HTTP_PORT="8090"

node {

    stage('Initialize'){
        def dockerHome = tool 'Docker'
        def mavenHome  = tool 'Maven'
        def gradleHome  = tool 'Gradle'
        env.PATH = "${dockerHome}/bin:${mavenHome}/bin:${gradleHome}/bin:${env.PATH}"
        echo "Application path is given here: ${env.PATH} "
    }

    stage('Checkout') {
         checkout([$class: 'GitSCM',
             branches: [[name: '*/pipeline']],
             doGenerateSubmoduleConfigurations: false,
             extensions: [[$class: 'CleanCheckout']],
             submoduleCfg: [],
             userRemoteConfigs: [[credentialsId: 'GitCreds', url: 'https://github.com/rverma2-cse/demo-app.git']]
         ])
    }

    stage('Build'){
        bat "mvn clean install"
    }

    stage("Stop container if running"){
        stopContainerIfRunning(CONTAINER_NAME)
    }

    stage("Delete container if exists"){
        removeContainerIfExists(CONTAINER_NAME)
    }

    stage('Image Build'){
        imageBuild(CONTAINER_NAME, CONTAINER_TAG)
    }


    /*stage('Push to Docker Registry'){
        withCredentials([usernamePassword(credentialsId: 'DockerCreds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            pushToImage(CONTAINER_NAME, CONTAINER_TAG, USERNAME, PASSWORD)
        }
    }
    */

    stage('Run App'){
        runApp(CONTAINER_NAME, CONTAINER_TAG, DOCKER_HUB_USER, HTTP_PORT)
    }

}

def stopContainerIfRunning(containerName){
    try {
        echo "Stopping container $containerName if running"
         def stdout = powershell(returnStdout: true, script: """
                foreach ($container in docker ps -q --filter=name=$containerName) {
                	docker stop $container
                }
                """)
            println stdout
    } catch(error){}
}

def removeContainerIfExists(containerName){
    try {
            echo "Removing container $containerName if exists"
             def stdout = powershell(returnStdout: true, script: """
                    foreach ($container in docker ps -q --filter=name=$containerName) {
	                    docker container rm $container
                    }
                    """)
                println stdout
        } catch(error){}
}

def imageBuild(containerName, tag){
    bat "docker build -t $containerName:$tag  -t $containerName --pull --no-cache ."
    echo "Image build complete"
}

def pushToImage(containerName, tag, dockerUser, dockerPassword){
    bat "docker login -u $dockerUser -p $dockerPassword"
    bat "docker tag $containerName:$tag $dockerUser/$containerName:$tag"
    bat "docker push $dockerUser/$containerName:$tag"
    echo "Image push complete"
}

def runApp(containerName, tag, dockerHubUser, httpPort){
    bat "docker pull $dockerHubUser/$containerName:$tag"
    bat "docker run -d --rm -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
    echo "Application started on port: ${httpPort} (http)"
}