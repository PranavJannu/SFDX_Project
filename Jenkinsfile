import groovy.json.JsonSlurperClassic

node {
    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME
    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH

    println 'KEY IS'
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println CONNECTED_APP_CONSUMER_KEY
    def toolbelt = tool 'toolbelt'

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        // Email Approval Stage
        stage('Email Approval') {
            // Send email notification for manual approval
            emailext (
                to: 'pranavjannu6@gmail.com',
                subject: 'Approval Needed: Deploy to Production',
                contentType: 'text/html', // Set content type to HTML
                body: """
                    <html>
                    <body>
                        <p>Please approve the deployment to production by clicking the link below:</p>
                        <p><a href="${BUILD_URL}/input/Proceed-to-Deployment/approve">Approve Deployment</a></p>
                    </body>
                    </html>
                """
            )
        }

        // Manual Approval Stage
        stage('Manual Approval') {
            // Pause pipeline execution until approval is received
            input(message: "Deploy to Production?", ok: "Approve", parameters: [[$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Do you approve the deployment to production?', name: 'Approved']])
        }

        // Deploy Code Stage
        stage('Deploy Code') {
            // Your deployment steps here
            if (isUnix()) {
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            } else {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }
            if (rc != 0) { error 'hub org authorization failed' }

            println rc

            // Deployment Command
            if (isUnix()) {
                rmsg = sh returnStdout: true, script: "${toolbelt} force:source:deploy --manifest manifest/package.xml -u ${HUB_ORG}"
            } else {
                rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:source:deploy --manifest manifest/package.xml -u ${HUB_ORG}"
            }

            // Display Deployment Output
            println('Deployment Output:')
            println(rmsg)
        }
    }
}
