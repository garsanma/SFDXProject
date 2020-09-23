#!groovy
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
	
    def command(script) {
    if (isUnix()) {
        return sh(returnStatus: true, script: script);
    } else {
        return bat(returnStatus: true, script: script);
    }	
	    
try{	

	
    stage('Checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
	// -------------------------------------------------------------------------
        // Authorize the Dev Hub org with JWT key 
        // -------------------------------------------------------------------------
        stage('Authorize') {
            //if (isUnix()) {
		    rc = command "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            //    rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
           // }else{
             //    rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }
            if (rc != 0) { error 'hub org authorization failed' }
	}	
	// -------------------------------------------------------------------------
        // Create new scratch org to test your code.
        // -------------------------------------------------------------------------
        stage('Create Test Scratch Org') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:org:create   --definitionfile config/project-scratch-def.json --wait 10 --durationdays 1"
                if (rc != 0) {
                    error 'Salesforce test scratch org creation failed.'
                }
        }    
	stage('Deploye Code') {
			// need to pull out assigned username
			if (isUnix()) {
				rmsg = sh returnStdout: true, script: "${toolbelt} force:mdapi:deploy -d manifest/. -u ${HUB_ORG} -w10"
			}else{
			   rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:mdapi:deploy -d manifest/. -u ${HUB_ORG} -w10"
			}
	    println 'Resultado deploy'		
			  
            println rmsg
            println('Hello from a Job DSL script!')
            //println(rmsg)
        }
    }
} catch (e){
        throw e
}finally {
        //cleanWs()
	stage ('Logout'){
		if (isUnix()) {rc = sh returnStatus: true, script: "\"${toolbelt}\" force:auth:logout -u ${HUB_ORG} -p" }
		else {rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:logout -u ${HUB_ORG} -p" }
		if (rc != 0) { error 'Error logout' }
	}
    }
}

