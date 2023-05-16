node
{
def mavenHome = tool name: "maven_software"
properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '5', numToKeepStr: '10')), [$class: 'JobLocalConfiguration', changeReasonComment: ''], pipelineTriggers([pollSCM('* * * * *')])])


echo "The Job name is: ${env.JOB_NAME}"
echo "The node name is: ${env.NODE_NAME}"
echo "The workspace path is: ${env.WORKSPACE}"
echo "The node label is: ${env.NODE_LABELS}"
echo "The build number is: ${env.BUILD_NUMBER}"
stage('Checkout the code from github'){
git credentialsId: 'Github_repo', url: 'https://github.com/HarshaVenkatTechnologies/maven-web-application.git'
}
stage('Building a artifactory'){
sh "${mavenHome}/bin/mvn clean package" 
}
stage('Generating a sonarqube report'){
sh "${mavenHome}/bin/mvn sonar:sonar"

}
stage ('Upload artifactory'){
sh "${mavenHome}/bin/mvn deploy"
}
stage ('deploying to tomcat server'){
sshagent(['67f17b7b-d810-4497-a895-d66a8cff52b9']) {
   sh "scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/Jenkins_pipeline/target/Govardhan-Technologies.war ec2-user@3.136.19.22:/opt/tomcat9/webapps/"
}

}
stage('Executing a shell'){
sh '''
            #!/bin/bash
            echo "Hello, world!"
            echo "This is a shell script executed in Jenkins."
            # Add your shell commands here
            # ...
        '''
}

}// node is closing

def notifySlack(String buildStatus = 'STARTED') {
    
    buildStatus = buildStatus ?: 'SUCCESS'

    def color

    if (buildStatus == 'STARTED') {
        color = '#D4DADF'
    } else if (buildStatus == 'SUCCESS') {
        color = '#BDFFC3'
    } else if (buildStatus == 'UNSTABLE') {
        color = '#FFFE89'
    } else {
        color = '#FF9FA1'
    }

    def msg = "${buildStatus}: `${env.JOB_NAME}` #${env.BUILD_NUMBER}:\n${env.BUILD_URL}"

    slackSend(color: color, message: msg)
}

node {
    try {
        notifySlack()

        // Existing build steps.
    } catch (e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        notifySlack(currentBuild.result)
    }
}
