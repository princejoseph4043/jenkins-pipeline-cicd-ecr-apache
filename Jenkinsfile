node {
    def imageTag        = "122910936396.dkr.ecr.us-east-1.amazonaws.com/hello-world"
    def serviceName     = "hello-world-service"
    def taskFamily      = "hello-world"
    def clusterName     = "ecscluster-ECSCluster-ti1zVZcvYlpN"
    def remoteImageTag  = "${imageTag}_${BUILD_NUMBER}"
    def taskDefile      = "file://aws/task-definition-${remoteImageTag}.json"
    def ecRegistry      = "https://122910936396.dkr.ecr.us-east-1.amazonaws.com"

    stage('Clone repository') {
        git branch: "master", url: "https://github.com/princejoseph4043/jenkins-freestyle-cicd-ecr-apache.git"
    }

    stage('Build image') {
        sh "docker build -t ${remoteImageTag} ."
    }

    stage('Push image') {
        docker.withRegistry('ecRegistry', "ecr:us-east-1:ecr-authentication") {
          docker.image("${remoteImageTag}").push(remoteImageTag)
        }
    }
}
