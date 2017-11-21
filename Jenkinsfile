#!groovy
import groovy.json.JsonSlurperClassic

node('master') {

    def CONSUMER_KEY='3MVG9g9rbsTkKnAXZvk2_HpxirjuGT1ZYP3SEUc3vn81bRJuFP2x_7rYo3huC_Fgr0TH0XBL8yQGrutjP8TAF'
    def JWT_KEY_FILE='/var/jenkins_home/certificates/server2.key'
    def HUB_USERNAME='nirish.okram@ghub1.com'
    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    
    // sh "sfdx force:auth:jwt:grant --clientid ${CONSUMER_KEY} --username ${HUB_USERNAME} --jwtkeyfile ${JWT_KEY_FILE} --setdefaultdevhubusername"

    try{

        stage('checkout') {
  				checkout([$class: 'GitSCM', branches: [[name: 'master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [],
  					userRemoteConfigs: [[url: 'git@github.com:okram999-sfdx/sfdx-travisci.git']]])
        }
        
        withCredentials([file(credentialsId: '82d3cfc9-674d-4e94-9aed-e7acaecee3f2', variable: 'server_key')]) {
            
            stage('Create Scratch Org') {
                rc = sh returnStatus: true, script: "sfdx force:auth:jwt:grant --clientid ${CONSUMER_KEY} --username ${HUB_USERNAME} --jwtkeyfile ${server_key} --setdefaultdevhubusername"
                    if (rc != 0) { error 'hub org authorization failed' }

                // create scratch org
                rmsg = sh returnStdout: true, script: "sfdx force:org:create -s -f config/project-scratch-def.json --json -a ciorg"
                printf rmsg
                def jsonSlurper = new JsonSlurperClassic()
                def robj = jsonSlurper.parseText(rmsg)
                if (robj.status != "ok") { error 'org creation failed: ' + robj.message }
                // SFDC_USERNAME=robj.username
                robj = null
            }
        
            stage('Push To Test Org') {
                rc = sh returnStatus: true, script: "sfdx force:source:push -u ciorg"
                if (rc != 0) {
                   error 'source push failed'
                }
                // assign permset
                // rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:user:permset:assign --targetusername ${SFDC_USERNAME} --permsetname WIPDeveloper"
                // if (rc != 0) {
                //     error 'permset:assign failed'
                // }
            }
            
            stage('Run Apex Test') {
                sh "mkdir -p ${RUN_ARTIFACT_DIR}"
                timeout(time: 120, unit: 'SECONDS') {
                rc = sh returnStatus: true, script: "sfdx force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap -u ciorg"
                if (rc != 0) {
                    error 'apex test run failed'
                        }
                }
            }

            stage('collect results') {
                junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
            }
        
        
        }


    }

    catch(err){
        println "sending email..../slack"
    }

}