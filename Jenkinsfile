import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  stage('init') {
    checkout scm
  }
  
  stage('build') {
    sh 'mvn clean package'
  }
  
  stage('deploy') {
    def resourceGroup = 'jenkins-rg' 
    def webAppName = 'JavaSampleJenkins'
    // login Azure
    withCredentials([azureServicePrincipal('azsrvsp')]) {
      sh '''
        az login --service-principal -u 3cb9f9a1-fdf8-4692-b7f3-08ab93802d5f -p Pu4l_5cMYf2f0p_4PQ_7os..-dwJpb6WYS -t 72f988bf-86f1-41af-91ab-2d7cd011db47
        az account set -s 7b886822-bfaf-4ba7-9df9-0f5481616177
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
