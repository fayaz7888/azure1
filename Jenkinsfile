import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=483c8e70-bebd-496e-b774-d5d504463981',
        'AZURE_TENANT_ID=a0986b17-38e7-43d4-95ee-1f86d69790cd']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'pcappservicelinuxrg'
      def webAppName = 'fayaz12'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'ee3276ce-892c-4620-8a76-b2b1a88bfe06', passwordVariable: 'hi58Q~5Q2uXC4a~iR8Lxq9ZF0dQi9cgM-zJhZa5b', usernameVariable: '3812a865-39f5-410b-8751-8f534dd41d80')]) {
       sh '''
          az login --service-principal -u 3812a865-39f5-410b-8751-8f534dd41d80 -p hi58Q~5Q2uXC4a~iR8Lxq9ZF0dQi9cgM-zJhZa5b -t d2a718e9-50aa-40bd-8f7d-68e0b9cce618
          az account set -s 483c8e70-bebd-496e-b774-d5d504463981

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
