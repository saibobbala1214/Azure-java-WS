import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=d1358f46-89c7-4569-8118-478a19adc81c',
        'AZURE_TENANT_ID=3e37509a-33ca-4c08-aa44-72f960edfb76']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'workshop-Azure_group'
      def webAppName = 'sai-app1'
      // login Azure
      //withCredentials([usernamePassword(credentialsId: 'jenkins-azure', passwordVariable: 'Lxe8Q~uwONB_qR3l1iM7~UzGh~ZhauknNul26c2a', usernameVariable: 'b17a2419-eb07-4632-a495-e85cb5f09170')]) {
      withCredentials([usernamePassword(credentialsId: 'jenkins-azure', passwordVariable: 'rK.8Q~sibBfgZOanzoCEyrIfCHt0LcM9LpWDOdBX', usernameVariable: 'b17a2419-eb07-4632-a495-e85cb5f09170')]) {
       sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID

          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
