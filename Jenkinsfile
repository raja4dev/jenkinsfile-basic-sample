node {
    def server
    def buildInfo
    def rtMaven
    stage (‘Clone’) {
        git url: "https://github.com/raja4dev/javasample.git"
    }
    stage (‘Artifactory configuration’) {
        // Obtain an Artifactory server instance, defined in Jenkins --> Manage:
        server = Artifactory.server ‘rajaweek5xray’
        rtMaven = Artifactory.newMavenBuild()
        rtMaven.tool = ‘Maven’ // Tool name from Jenkins configuration
        rtMaven.deployer releaseRepo: ‘libs-release-local’, snapshotRepo: ‘libs-snapshot-local’, server: server
        rtMaven.resolver releaseRepo: ‘libs-release’, snapshotRepo: ‘libs-snapshot’, server: server
        rtMaven.deployer.deployArtifacts = false // Disable artifacts deployment during Maven run
        buildInfo = Artifactory.newBuildInfo()
    }
    stage (‘Install’) {
        rtMaven.run pom: ‘MavenProject/pom.xml’, goals: ‘clean install’, buildInfo: buildInfo
    }
    stage (‘Deploy’) {
        rtMaven.deployer.deployArtifacts buildInfo
    }
    stage (‘Publish build info’) {
        server.publishBuildInfo buildInfo
    }
    stage(‘XRay Scan’) {
        def scanConfig = [
            ‘buildName’ : buildInfo.name,
            ‘buildNumber’: buildInfo.number,
            ‘failBuild’: false
        ]
        def scanResult = server.xrayScan scanConfig
        echo scanResult as String
   }
    stage (‘Promotion’) {
    promotionConfig=[
        ‘buildName’ : buildInfo.name,
        ‘buildNumber’: buildInfo.number,
        ‘targetRepo’ :‘libs-release-local’,
        ‘comment’ : ‘Promoting to Release’,
        ‘sourceRepo’ : ‘libs-snapshot-local’,
        ‘status’ : ‘Released’,
        ‘includeDependencies’: true,
        ‘failFast’ : false,
        ‘copy’ : true
        ]
        server.promote promotionConfig
    }
}
