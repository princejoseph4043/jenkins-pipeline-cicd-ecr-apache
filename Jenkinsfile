node {
    def app

    stage('Clone repository') {
        git branch: "master", url: "https://github.com/princejoseph4043/jenkins-freestyle-cicd-ecr-apache.git"
    }

    stage('Build image') {
        sh "docker build -t 122910936396.dkr.ecr.us-east-1.amazonaws.com/hello-world:v_$BUILD_NUMBER ."
    }

    stage('Push image') {
        docker.withRegistry('https://122910936396.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:ecr-authentication') {
            sh "docker push 122910936396.dkr.ecr.us-east-1.amazonaws.com/hello-world:v_$BUILD_NUMBER"
        }
    }
}
