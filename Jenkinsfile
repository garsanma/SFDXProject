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
            if (isUnix()) {
		rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }else{
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }
            if (rc != 0) { error 'hub org authorization failed' }
	}	

	stage('Source Deploy') {
		
		
		   
			// need to pull out assigned username
			if (isUnix()) {				
				rc = sh returnStdout: true, script: "${toolbelt} force:source:deploy -x ./manifest/package.xml -u ${HUB_ORG}"
			}else{
				//rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:deploy  -x ./manifest/package.xml -u ${HUB_ORG}"
				//rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:deploy  -x ./manifest/package.xml -u ${HUB_ORG}"
				rc = command "\"${toolbelt}\" force:source:deploy  -x ./manifest/package.xml -u ${HUB_ORG}"
			}
	    if (rc != 0) { error 'Error Deploy' }	
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

def command(script) {
    if (isUnix()) {
        return sh(returnStatus: true, script: script);
    } else {
        return bat(returnStatus: true, script: script);
    }
}


