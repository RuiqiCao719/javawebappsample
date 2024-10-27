import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles) {
    if (p['publishMethod'] == 'FTP') {
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
    }
  }
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID= 2c6f72d0-1a7d-4a5c-b4b7-c53f7dcec25f',
           'AZURE_TENANT_ID= e0422109-93f9-4ada-897e-3d48f548adc2']) {
    stage('init') {
      checkout scm
    }

    stage('build') {
      sh 'mvn clean package'
    }

    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = 'jenkinsrqc'
      // login to Azure
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable:'uNR8Q~xrvQsI.2whDPJG1Xs5xl38rdSe.YFZxbOST', usernameVariable: '7fbba59e-c10e-42c4-aad7-785c63df8eaf')]) {
        sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile(pubProfilesJson)
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
