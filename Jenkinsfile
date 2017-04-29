#!groovy

properties([
  buildDiscarder(
    logRotator(
      daysToKeepStr: '10',
      numToKeepStr: '7'
    )
  )
]);

node {
  try {
    stage('Checkout') {
      checkout scm
      env.GIT_COMMIT = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
    }
    stage('Installation') {
      sh 'npm install'
    }
    stage('Code quality') {
      sh 'gulp all:lint'
      sh 'gulp public:lint'
      sh 'gulp integration:lint'

      if (env.CHANGE_ID) {
        def scannerHome = tool name: 'SonarQube Scanner 2.8',
          type: 'hudson.plugins.sonar.SonarRunnerInstallation'
        withSonarQubeEnv('CD-SonarQube') {
          withCredentials([
            string(credentialsId: 'sonarGithubOauth', variable: 'SONAR_GITHUB_OAUTH')
          ]) {
            sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=sonar.properties \
              -Dsonar.projectVersion=${env.CHANGE_ID}.${env.GIT_COMMIT} \
              -Dsonar.github.pullRequest=${env.CHANGE_ID} \
              -Dsonar.github.oauth=${SONAR_GITHUB_OAUTH}"
          }
        }
      }
    }
    stage('Backend test') {
      env.NODE_ENV_TMP=env.NODE_ENV
      env.NODE_ENV = sh(returnStdout: true, script: 'date +"%Y%m%m_%H%M%S_%N"').trim()
      env.AUTH_PORT = sh(returnStdout: true, script: 'date +"2%S%1N"').trim()
      env.CATALOG_PORT = sh(returnStdout: true, script: 'date +"3%S%1N"').trim()
      env.CUSTOMIZATION_PORT = sh(returnStdout: true, script: 'date +"4%S%1N"').trim()
      env.INGESTION_PORT = sh(returnStdout: true, script: 'date +"5%S%1N"').trim()
      env.NOTIFICATION_PORT = sh(returnStdout: true, script: 'date +"6%S%1N"').trim()
      env.ORDER_PORT = sh(returnStdout: true, script: 'date +"7%S%1N"').trim()
      
      ansiColor('xterm') {
        sh 'gulp auth:server:coverage'
        sh 'gulp catalog:server:coverage'
        sh 'gulp cds-common:coverage'
        sh 'gulp customization:server:coverage'
        sh 'gulp ingestion:server:coverage'
        sh 'gulp notification:server:coverage'
        sh 'gulp order:server:coverage'
      }

      env.NODE_ENV=env.NODE_ENV_TMP

      publishHtmlReport('reports/backend-coverage/auth', 'Auth coverage')
      publishHtmlReport('reports/backend-coverage/customization', 'Customization coverage')
      publishHtmlReport('reports/backend-coverage/catalog', 'Catalog coverage')
      publishHtmlReport('reports/backend-coverage/ingestion', 'Ingestion coverage')
      publishHtmlReport('reports/backend-coverage/notification', 'Notification coverage')
      publishHtmlReport('reports/backend-coverage/order', 'Order coverage')
    }
    stage('Frontend test') {
      env.PUBLIC_PORT = sh(returnStdout: true, script: 'date +"8%S%1N"').trim()
      env.SELENIUM_PORT = sh(returnStdout: true, script: 'date +"9%S%1N"').trim()
      env.WEB_SOCKET_PORT = sh(returnStdout: true, script: 'date +"6%S%1N"').trim()
      
      ansiColor('xterm') {
        sh 'gulp protractor-update'
        sh 'gulp public:test'
      }

      publishHtmlReport('reports/frontend-tests', 'Frontend coverage')
    }
  } catch (e) {
    def slackMessage = "${env.JOB_NAME} - #${env.BUILD_NUMBER} Failure: <${env.BUILD_URL}console|Details>"

    if (env.CHANGE_ID) {
      slackMessage = "PR #${env.CHANGE_ID} Failure: <${env.BUILD_URL}console|Details>"
    }

    slackSend(color: 'danger', message: slackMessage)

    throw e
  }
}

/*
 * Publish html reports in the job
 */
def publishHtmlReport(reportDir, reportName) {
  publishHTML([
    allowMissing: true,
    alwaysLinkToLastBuild: true,
    keepAll: true,
    reportDir: "${reportDir}",
    reportFiles: 'index.html',
    reportName: "${reportName}"
  ])
}

def sendSlackMessage() {
  
}
