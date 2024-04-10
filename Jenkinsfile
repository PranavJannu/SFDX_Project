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
        stage('Email Approval') {
            emailext (
                to: 'duser1273@gmail.com',
                subject: 'Approval Needed: Deploy to Production',
                body: 'Please approve the deployment to production by clicking the button below:',
                replyTo: 'jenkins@example.com', // Optional: Specify a reply-to address
                attachmentsPattern: 'path/to/attachment/files/*.txt', // Optional: Attach files if needed
                buttons: [
                    [$class: 'ApproveButton', text: 'Approve', url: '$BUILD_URL/input/Proceed-to-Deployment/approve'],
                    [$class: 'RejectButton', text: 'Reject', url: '$BUILD_URL/input/Proceed-to-Deployment/reject']
                ]
            )
        }

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
