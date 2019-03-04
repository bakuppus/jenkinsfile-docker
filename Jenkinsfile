pipeline {
   agent any

   stages {
       stage('Artifactory') {
        steps {

            script {
              node {

                // Cleanup workspace
                deleteDir()

                //copy war
                sh "cp -rf /var/lib/jenkins/workspace/jenkinsfile/target/javaee7-simple-sample.war ."

                def buildInfo = Artifactory.newBuildInfo()
                def tagDockerApp

                //Clone example project from GitHub repository
                git url: 'https://github.com/bakuppus/jenkinsfile-docker.git', branch: 'master'

                // Step 1: Obtain an Artifactiry instance, configured in Manage Jenkins --> Configure System:
                def server = Artifactory.server 'Jfrog'

                // Step 2: Create an Artifactory Docker instance:
                def rtDocker = Artifactory.docker server: server
                // Or if the docker daemon is configured to use a TCP connection:
                // def rtDocker = Artifactory.docker server: server, host: "tcp://<docker daemon host>:<docker daemon port>"
                // If your agent is running on OSX:
                // def rtDocker= Artifactory.docker server: server, host: "tcp://127.0.0.1:1234"
                tagDockerApp = "18.219.249.212:8081/kierandocker/tomcat8:latest"
                docker.build(tagDockerApp)

                server.bypassProxy = true
                // Step 3: Push the image to Artifactory.
                // Make sure that <artifactoryDockerRegistry> is configured to reference <targetRepo> Artifactory repository. In case it references a different repository, your build will fail with "Could not find manifest.json in Artifactory..." following the push.
                buildInfo = rtDocker.push "$tagDockerApp", 'kierandocker'

                // Step 4: Publish the build-info to Artifactory:
                server.publishBuildInfo buildInfo

                /**
                 * Send notifications based on build status string
                 */
                def call(String buildStatus = 'STARTED') {
                  // build status of null means successful
                  buildStatus = buildStatus ?: 'SUCCESS'

                  // Default values
                  def colorName = 'RED'
                  def colorCode = '#FF0000'
                  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
                  def summary = "${subject} (${env.BUILD_URL})"
                  def details = """<p>${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                    <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>"""

                  // Override default values based on build status
                  if (buildStatus == 'STARTED') {
                    color = 'YELLOW'
                    colorCode = '#FFFF00'
                  } else if (buildStatus == 'SUCCESS') {
                    color = 'GREEN'
                    colorCode = '#00FF00'
                  } else {
                    color = 'RED'
                    colorCode = '#FF0000'
                  }

                  // Send notifications
                  slackSend (color: colorCode, message: summary)

                  hipchatSend (color: color, notify: true, message: summary)

                  emailext (
                      to: 'bitwiseman@bitwiseman.com',
                      subject: subject,
                      body: details,
                      recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                    )
                }
                

            }
        }
      }
     }





    }
  }
