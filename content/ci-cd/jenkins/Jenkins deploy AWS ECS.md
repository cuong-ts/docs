# Jenkins deploy AWS ECS

Jenkins deploy AWS ECS

{% code title="Jenkinsfile" %}
```groovy
node {
  ws("workspace/${env.JOB_NAME}/${env.BRANCH_NAME}") {
    try {

      def imageTag
      def serviceName
      def taskFamily
      def dockerFilePrefix
      def clusterName
      def envName

      if  (env.BRANCH_NAME == "dev") {
        imageTag          = ""
        serviceName       = ""
        taskFamily        = ""
        dockerFilePrefix  = ""
        clusterName       = ""
        envName           = ""
      } 
      
      if  (env.BRANCH_NAME == "prod") {
        imageTag          = ""
        serviceName       = ""
        taskFamily        = ""
        dockerFilePrefix  = ""
        clusterName       = ""
        envName           = ""
      } 
      

      // Notify slack, new build started!
      stage("BUILD STARTED") {
        slackSend (username: 'CI', channel: "slack-channel", color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }


      def remoteImageTag  = "${imageTag}:${BUILD_NUMBER}"
      def taskDefile      = "file://aws/task-definition-${BUILD_NUMBER}.json"
      def ecRegistry      = "https://your-aws-acc.dkr.ecr.ap-southeast-1.amazonaws.com"
      def nodeHome = tool name: "nodejs13",
                          type: "jenkins.plugins.nodejs.tools.NodeJSInstallation"
      env.PATH = "${nodeHome}/bin:${env.PATH}"


      stage("Checkout") {
        checkout scm
      }

      stage("Docker build") {

        sh "docker build -t your-aws-acc.dkr.ecr.ap-southeast-1.amazonaws.com/${remoteImageTag} \
                                    -f ${dockerFilePrefix}.Dockerfile ."
      }

      stage("Docker push") {

          sh "aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin your-aws-acc.dkr.ecr.ap-southeast-1.amazonaws.com/${remoteImageTag}"
          sh "docker push your-aws-acc.dkr.ecr.ap-southeast-1.amazonaws.com/${remoteImageTag}"
      }

      stage("Deploy") {

        sh "cp -f /var/jenkins_home/workspace/${clusterName}_td.json aws/task-definition.json"
        sh  "                                                                     \
          sed -e  's;%BUILD_TAG%;${BUILD_NUMBER};g'                             \
                  aws/task-definition.json >                                      \
                  aws/task-definition-${BUILD_NUMBER}.json                      \
        "


        // Register the new [TaskDefinition]
        sh  "                                                                     \
          aws ecs register-task-definition  --family ${taskFamily}                \
                                            --cli-input-json ${taskDefile}        \
                                            > /dev/null                           \
        "

        // Get the last registered [TaskDefinition#revision]
        def taskRevision = sh (
          returnStdout: true,
          script:  "                                                              \
            aws ecs describe-task-definition  --task-definition ${taskFamily}     \
                                              | egrep 'TASKDEFINITION'                  \
                                              | awk '{print \$4}'                 \
          "
        ).trim()

        // ECS update service to use the newly registered [TaskDefinition#revision]
        //
        sh  "                                                                     \
          aws ecs update-service  --cluster ${clusterName}                        \
                                  --service ${serviceName}                        \
                                  --task-definition ${taskFamily}:${taskRevision} \
                                  --desired-count 1                               \
                                  --force-new-deployment                          \
        "
      }

      stage("BUILD SUCCEED") {
        slackSend (username: 'CI', channel: "slack-channel", color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
    } catch(e) {
      slackSend (username: 'CI', channel: "slack-channel", color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      throw e
    }
  }
}
```
{% endcode %}



