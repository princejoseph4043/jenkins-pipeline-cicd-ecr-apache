node {
    def imageTag        = "hello-world"
    def ecRegistry      = "https://122910936396.dkr.ecr.us-east-1.amazonaws.com"
    def remoteImageTag  = "v_${BUILD_NUMBER}"
    def serviceName     = "hello-world-service"
    def taskFamily      = "hello-world"
    def clusterName     = "ecscluster-ECSCluster-ti1zVZcvYlpN"
    def taskDefile      = "file://aws/task-definition-${remoteImageTag}.json"

    stage('Clone repository') {
        git branch: "master", url: "https://github.com/princejoseph4043/jenkins-pipeline-cicd-ecr-apache.git"
    }

    stage('Build image') {
        sh "docker build -t hello-world:${remoteImageTag} ."
    }
    
    stage("Docker push") {
    docker.withRegistry(ecRegistry, "ecr:us-east-1:ecr-authentication") {
          docker.image("hello-world:${remoteImageTag}").push(remoteImageTag)
    }
}
    stage("Deploy") {
        // Replace BUILD_TAG placeholder in the task-definition file -
        // with the remoteImageTag (imageTag-BUILD_NUMBER)
        sh  "                                                                     \
          sed -i  's/%BUILD_TAG%/${remoteImageTag}/g'                             \
                  aws/task-definition.json >                                      \
                  aws/task-definition-${remoteImageTag}.json                      \
        "

        // Get current [TaskDefinition#revision-number]
        def currTaskDef = sh (
          returnStdout: true,
          script:  "                                                              \
            aws ecs describe-task-definition  --task-definition ${taskFamily}     \
                                              | egrep 'revision'                  \
                                              | tr ',' ' '                        \
                                              | awk '{print \$2}'                 \
          "
        ).trim()

        def currentTask = sh (
          returnStdout: true,
          script:  "                                                              \
            aws ecs list-tasks  --cluster ${clusterName}                          \
                                --family ${taskFamily}                            \
                                --output text                                     \
                                | egrep 'TASKARNS'                                \
                                | awk '{print \$2}'                               \
          "
        ).trim()

        /*
        / Scale down the service
        /   Note: specifiying desired-count of a task-definition in a service -
        /   should be fine for scaling down the service, and starting a new task,
        /   but due to the limited resources (Only one VM instance) is running
        /   there will be a problem where one container is already running/VM,
        /   and using a port(80/443), then when trying to update the service -
        /   with a new task, it will complaine as the port is already being used,
        /   as long as scaling down the service/starting new task run simulatenously
        /   and it is very likely that starting task will run before the scaling down service finish
        /   so.. we need to manually stop the task via aws ecs stop-task.
        */
        if(currTaskDef) {
          sh  "                                                                   \
            aws ecs update-service  --cluster ${clusterName}                      \
                                    --service ${serviceName}                      \
                                    --task-definition ${taskFamily}:${currTaskDef}\
                                    --desired-count 0                             \
          "
        }
        if (currentTask) {
          sh "aws ecs stop-task --cluster ${clusterName} --task ${currentTask}"
        }

        // Register the new [TaskDefinition]
        sh  "                                                                     \
          aws ecs register-task-definition  --family ${taskFamily}                \
                                            --cli-input-json file://${taskDefile}        \
        "

        // Get the last registered [TaskDefinition#revision]
        def taskRevision = sh (
          returnStdout: true,
          script:  "                                                              \
            aws ecs describe-task-definition  --task-definition ${taskFamily}     \
                                              | egrep 'revision'                  \
                                              | tr ',' ' '                        \
                                              | awk '{print \$2}'                 \
          "
        ).trim()

        // ECS update service to use the newly registered [TaskDefinition#revision]
        //
        sh  "                                                                     \
          aws ecs update-service  --cluster ${clusterName}                        \
                                  --service ${serviceName}                        \
                                  --task-definition ${taskFamily}:${taskRevision} \
                                  --desired-count 1                               \
        "
      }    
  }
