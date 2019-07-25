properties([
    gitLabConnection('gilabus'),
    pipelineTriggers([
        [
            $class: 'GitLabPushTrigger',
            branchFilterType: 'All',
            triggerOnPush: false,
            triggerOnMergeRequest: false,
            triggerOpenMergeRequestOnPush: "never",
            triggerOnNoteRequest: false,
            noteRegex: "Jenkins please retry a build",
            skipWorkInProgressMergeRequest: false,
            //secretToken: project_token,
            ciSkip: false,
            setBuildDescription: false,
            addNoteOnMergeRequest: false,
            addCiMessage: false,
            addVoteOnMergeRequest: false,
            acceptMergeRequestOnSuccess: false,
            branchFilterType: "NameBasedFilter",
            includeBranchesSpec: "release/qat",
            excludeBranchesSpec: "",
        ]
    ])
])

node('docker-maven-build-slave1') {
try {
    notifyStarted()
    
     if (env.BRANCH_NAME == 'master') {
            echo 'I only execute on the Master branch'
            stage('Checkout scm /master') {
            checkout scm
            } 
          //stage('SonarQube analysis') {
          //sh "mvn sonar:sonar"
            //} 
            stage('Build & Test') {
            def mvnHome = tool 'M3'
            sh "${mvnHome}/bin/mvn -B -Dmaven.test.failure.ignore verify"
            }
            stage('Uploading artifact to Nexus') {
            nexusArtifactUploader artifacts: [[artifactId: 'maven-deploy-plugin', classifier: '', file: '$nexusArtifact', type: 'war']], credentialsId: 'nexus', groupId: 'org.apache.maven.plugins', nexusUrl: '10.26.34.246:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'test1', version: '2.$BUILD_NUMBER'
            }
     } 
     if (env.BRANCH_NAME == 'develop') {
            echo 'I only execute on the develop branch'
            stage('Checkout scm /develop') {
            checkout scm
            } 
          //stage('SonarQube analysis') {
          //sh "mvn sonar:sonar"
            //} 
            stage('Build & Test') {
            def mvnHome = tool 'M3'
            sh "${mvnHome}/bin/mvn -B -Dmaven.test.failure.ignore verify"
            }
            stage('Archive artifact') {
            archiveArtifacts 'target/*.war'
            } 
            stash includes:
            'target/OMS.war,src/pt/Hello_world.jmx',
            name: 'binary'
  
            stage('Deploy to stage') {
            sshagent(['ssh_tomcat']) {
            sh '$COMMAND'
            }
            }
            stage ('Performance Testing'){
            // Execute command to exec performance testing with the use of Apache JMeter
            // Waiting for 20 seconds before start test(until tomcat deploy war-file)
            sh '''cd /home/jenkins/apache-jmeter-5.1.1/bin/
            sleep 15
            ./jmeter.sh -n -t $WORKSPACE/src/pt/Hello_world.jmx -l $WORKSPACE/test_report.jtl'''
            // Archive result of performance testing
            archiveArtifacts '**/*.jtl'
            // Publish Apache JMeter results
            //perfReport errorFailedThreshold: 5, errorUnstableThreshold: 3, percentiles: '0,50,90,100', sourceDataFiles: '**/*.jtl'
            }
      } 
notifySuccessful()
  }
  
  catch (e) {
    currentBuild.result = "FAILED"
    notifyFailed()
    throw e
  }
} 
def notifyStarted() { emailext (
      subject: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
      to: "$MyEmail",
      from: "$fromEmail"
    ) }

def notifySuccessful() { emailext (
      subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
      to: "$MyEmail",
      from: "$fromEmail"
    ) }

def notifyFailed() {emailext (
      subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
      to: "$MyEmail",
      from: "$fromEmail"
    )
}
