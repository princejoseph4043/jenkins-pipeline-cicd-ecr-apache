node {
    def imageTag        = "hello-world"
    def ecRegistry      = "https://122910936396.dkr.ecr.us-east-1.amazonaws.com"
    def remoteImageTag  = "v_${BUILD_NUMBER}"
    def serviceName     = "hello-world-service"
    def taskFamily      = "hello-world"
    def clusterName     = "ecscluster-ECSCluster-ti1zVZcvYlpN"
    def taskDefile      = "file://aws/task-definition-${remoteImageTag}.json"

    stage('Clone repository') {
        git branch: "master", url: "https://github.com/princejoseph4043/jenkins-freestyle-cicd-ecr-apache.git"
    }

    stage('Build image') {
        sh "docker build -t hello-world:${remoteImageTag} ."
    }
    
    stage("Docker push") {
    docker.withRegistry(ecRegistry, "ecr:us-east-1:ecr-authentication") {
          docker.image("hello-world:${remoteImageTag}").push(remoteImageTag)
    }
}
    
}
