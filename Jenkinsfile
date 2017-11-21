#!groovy
import groovy.json.JsonSlurperClassic

node('master') {

    def CONSUMER_KEY='3MVG9g9rbsTkKnAXZvk2_HpxirjuGT1ZYP3SEUc3vn81bRJuFP2x_7rYo3huC_Fgr0TH0XBL8yQGrutjP8TAF'
    def JWT_KEY_FILE='/var/jenkins_home/certificates/server2.key'
    def HUB_USERNAME='nirish.okram@ghub1.com'
    
    sh "sfdx force:auth:jwt:grant --clientid ${CONSUMER_KEY} --username ${HUB_USERNAME} --jwtkeyfile ${JWT_KEY_FILE} --setdefaultdevhubusername"

    try{

        stage('ci') {
            
            sh '''sfdx force:org:create -s -f config/project-scratch-def.json -a ciorg
                  sfdx force:source:push -u ciorg
                  sfdx force:apex:test:run -u ciorg -c -r human
                  sfdx force:org:delete -u ciorg -p
                '''  
        }


    }

    catch(err){

    }

}