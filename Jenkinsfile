node {
  stage ('Checkout'){
checkout scm
}
  stage('SonarQube analysis') {
                    // requires SonarQube Scanner 2.8+
                    //sh "npm install"
                    def scannerHome = tool 'Sonar-Scanner';
                    withSonarQubeEnv('SonarQube') {
                    def sample=env.JOB_NAME.replaceAll('/','.')
                    def projectKey=sample.replaceAll('%2F','.')
		    sh "${scannerHome}/bin/sonar-scanner -D sonar.host.url='http://3.110.155.212:9000' -D sonar.projectKey=${projectKey}  -D sonar.sources=. -D sonar.exclusions=node_modules/**,Dockerfile,docker-compose.yml,default.conf"
                     //stash includes: ".scannerwork/report-task.txt", name: 'sonar'
                    }
stage("Quality Gate"){
                    unstash 'sonar'
                    def props = getProperties(".scannerwork/report-task.txt")
                    def sonarServerUrl=props.getProperty('serverUrl')
                    def ceTaskUrl= props.getProperty('ceTaskUrl')
                    def ceTask
                    def analysisStatus
                    while(analysisStatus != 'SUCCESS') {
                        sleep(5)
                        ceTask = jsonParse(new URL(ceTaskUrl).getText(requestProperties: [Authorization: "Basic " + "admin:admin".getBytes().encodeBase64().toString()]))
                        analysisStatus = ceTask["task"]["status"]
                    }
                    def analysisId = ceTask["task"]["analysisId"]
                    def analysisResultUrl = sonarServerUrl + "/api/qualitygates/project_status?analysisId=" + analysisId
                    def qualitygate = jsonParse(analysisResultUrl.toURL().getText(requestProperties: [Authorization: "Basic " + "admin:admin".getBytes().encodeBase64().toString()]))
                    def qualitygateStatus = qualitygate["projectStatus"]["status"]
                    echo "qualitygateStatus=${qualitygateStatus}"
                    if ("ERROR".equals(qualitygateStatus)) {
                echo "Quality Gate FAILURE"
                emailext body: "Quality checks for the commit ${GIT_COMMIT_ID} !!!!!!!!!!!!!!!FAILED!!!!!!!!!!!!.. Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n Git committer email: ${GIT_COMMIT_EMAIL} and ${GIT_COMMIT_ID} \nServer IP",
                recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], subject: 'Pinpoint Quality Check Failure Notification ', to: "${emailID}"
               
                hangoutsNotify message: "Quality checks for the commit ${GIT_COMMIT_ID} !!!!!!!!!!!!!!!FAILED!!!!!!!!!!!!.. Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n Git committer email: ${GIT_COMMIT_EMAIL} and ${GIT_COMMIT_ID} \nServer IP ",token: "${chatToken}",threadByJob: false

            }
            
             timeout(time: 1, unit: 'MINUTES') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
             
         
	  
          }
}
