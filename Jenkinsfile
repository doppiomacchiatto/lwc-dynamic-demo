#!groovy

import groovy.json.JsonSlurperClassic

node{
    
    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_USERNAME=env.SF_USERNAME
    def SERVER_KEY_CREDENTIALS_ID=env.SERVER_KEY_CREDENTIALS_ID
    def JWT_KEY=env.JWT_KEY
    def TEST_LEVEL='ETA_App'
    def PACKAGE_NAME='ETA-Demo'
    def PACKAGE_VERSION='1.0.0'
    def ORG_ALIAS='dev4'
    def LOG_LEVEL='fatal'
    def SF_INSTANCE_URL = env.SF_INSTANCE_URL ?: "https://login.salesforce.com"

    // -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------

    stage('checkout source') {
        println "Test Print"
        println "server_key - ${SERVER_KEY_CREDENTIALS_ID}"
        checkout([$class: 'GitSCM', 
        branches: [[name: '*/main']], 
        userRemoteConfigs: 
            [[credentialsId: "${SERVER_KEY_CREDENTIALS_ID}", 
            url: 'https://gitlab.com/eq-dev-crew/eta-dynamic-demo.git']]])

    }
    withEnv(["HOME=${env.WORKSPACE}"]) {
        
        withCredentials([file(credentialsId: JWT_KEY, variable: 'server_key_file')]) {
            // -------------------------------------------------------------------------
            // Authorize the Dev org with JWT key and give it an alias.
            // -------------------------------------------------------------------------
            stage('auth-dev-org'){
                def rc = sh returnStatus: true, script: "sfdx auth:jwt:grant --instanceurl ${SF_INSTANCE_URL} --clientid ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwtkeyfile ${server_key_file} --setdefaultusername --setalias ${ORG_ALIAS}"
                if (rc != 0) {
                    error 'Salesforce Org authorization failed.'
                }
            }
            stage('deploy-source'){
                def rc = sh returnStatus: true, script: "sfdx force:source:deploy --sourcepath ./force-app/main/default --targetusername ${ORG_ALIAS} --loglevel ${LOG_LEVEL}"
                println "${rc}"
                if (rc != 0) {
                    error 'Salesforce Deplying Source.'
                }
            }
            stage('run tests'){
                def test = sh returnStatus: true, script: "sfdx force:apex:test:run --suitenames ${TEST_LEVEL} --wait 10 --resultformat tap --codecoverage --targetusername ${ORG_ALIAS} --outputdir assets/outputs"
                 if (test != 0) {
                    error 'Salesforce unit test run in test dev org failed.'
                }
            }
        }
    }
}