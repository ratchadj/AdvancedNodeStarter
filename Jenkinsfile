def CONTAINER_NAME="jenkins-pipeline"
def CONTAINER_TAG="latest"
def DOCKER_HUB_USER="ratchada.jududom@gmail.com"
def HTTP_PORT="8090"

node {

    stage('Initialize'){
        def dockerHome = tool 'myDocker'
        def nodejsHome  = tool 'myNodejs'
        env.PATH = "${dockerHome}/bin:${nodejsHome}/bin:${env.PATH}"
    }

    stage('Checkout') {
        checkout scm
    }

    stage('Build'){
        // sh "npm install --production"
    }
    
    stage('SonarQube analysis') {
        // requires SonarQube Scanner 2.8+
        def scannerHome = tool 'SonarQube Scanner';
        withSonarQubeEnv('SonarQube') {
          sh "${scannerHome}/bin/sonar-scanner " +
              "-Dsonar.projectKey=AdvancedNodeStarter:pipeline " +
              "-Dsonar.projectName=AdvancedNodeStarter-pipeline " +
              "-Dsonar.sources=. " +
              "-Dsonar.projectVersion=1.0 " +
              "-Dsonar.language=js " +
              "-Dsonar.sources=./ " +
              "-Dsonar.exclusions=node_modules/*" +
              "-Dsonar.sourceEncoding=UTF-8 "
        }
    }
    
    stage("Quality Gate") {
        script {
          timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
            def qualitygate = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
            if (qualitygate.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qualitygate.status}"
            }
          }
        }
    }
    
    stage('Test'){
        try {
            
        } catch(error){
            echo "The sonar server could not be reached ${error}"
        }
     }

    stage("Image Prune"){
        imagePrune(CONTAINER_NAME)
    }

    stage('Image Build'){
        imageBuild(CONTAINER_NAME, CONTAINER_TAG)
    }

    stage('Push to Docker Registry'){
        withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
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
