node {
    def notifyBuild = { String buildStatus ->
        // build status of null means successful
        buildStatus =  buildStatus ?: 'SUCCESSFUL'

        // Default values
        def colorName = 'RED'
        def colorCode = '#FF0000'
        def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
        def summary = "${subject} (${env.BUILD_URL})"
        def details = """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>"""

        // Override default values based on build status
        if (buildStatus == 'STARTED') {
            color = 'YELLOW'
            colorCode = '#FFFF00'
        } else if (buildStatus == 'SUCCESSFUL') {
            color = 'GREEN'
            colorCode = '#00FF00'
        } else {
            color = 'RED'
            colorCode = '#FF0000'
        }

        // Send notifications
        slackSend ( color: colorCode, message: summary)
    }



    stage ('Build') {
        try {
            notifyBuild('STARTED')

            dir("place") {
                sh 'mkdir bala'
            }
        } catch (e) {
            // If there was an exception thrown, the build failed
            currentBuild.result = "FAILED"
            throw e
        } finally {
            // Success or failure, always send notifications
            notifyBuild(currentBuild.result)
        }
    }
}
