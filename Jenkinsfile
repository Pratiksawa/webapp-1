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
		    sh "${scannerHome}/bin/sonar-scanner -D sonar.host.url='http://3.108.219.134:9000' -D sonar.projectKey=${projectKey}  -D sonar.sources=. -D sonar.exclusions=node_modules/**,Dockerfile,docker-compose.yml,default.conf"
                     //stash includes: ".scannerwork/report-task.txt", name: 'sonar'
                    }
 stage("Quality Gate Status Check"){
          sh 'sleep 20'
             def qg = waitForQualityGate()
             if (qg.status == 'OK') {
                echo "success"
             }     
     }
               
    }
}

