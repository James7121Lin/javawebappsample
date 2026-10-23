import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=<subscription_id>',
        'AZURE_TENANT_ID=<tenant_id>']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
    def resourceGroup = 'jenkins-get-started-rg'
    def webAppName = 'jenkins-get-started-james'

    withCredentials([usernamePassword(
        credentialsId: 'azure-service-principal',
        usernameVariable: 'AZURE_CLIENT_ID',
        passwordVariable: 'AZURE_CLIENT_SECRET'
    )]) {
        sh """
            az login --service-principal \
                -u "\$AZURE_CLIENT_ID" \
                -p "\$AZURE_CLIENT_SECRET" \
                -t "\$AZURE_TENANT_ID"

            az account set --subscription "\$AZURE_SUBSCRIPTION_ID"

            az webapp deploy \
                --resource-group "${resourceGroup}" \
                --name "${webAppName}" \
                --src-path target/calculator-1.0.war \
                --type war
        """
    }
}
