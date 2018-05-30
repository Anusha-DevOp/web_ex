// Obtaining an Artifactory server instance defined in Jenkins:
			
def server = Artifactory.server 'Artifactory Version 4.15.0'

		 //If artifactory is not defined in Jenkins, then create on:
		// def server = Artifactory.newServer url: 'Artifactory url', username: 'username', password: 'password'

//Create Artifactory Maven Build instance
def rtMaven = Artifactory.newMavenBuild()

def buildinfo

pipeline {
    agent any
    stages {
        stage('Clone sources'){
            steps {
                git url: 'https://github.com/Anusha-DevOp/web_ex'
            }
        }

     	stage('SonarQube analysis') {
	     steps {
		withMaven(maven : 'Maven-3.5.3') {
		   bat 'mvn clean package sonar:sonar'
		}
	      }
	}

	stage('Quality Gate') {
		steps {
			timeout(time: 1, unit: 'HOURS') {
			//Parameter indicates wether to set pipeline to UNSTABLE if Quality Gate fails
		        // true = set pipeline toUNSTABLE, false = don't
			// Requires SonarQube Scanner for Jenkins 2.7+

			waitForQualityGate abortPipeline: true
		       }
		 }
	}

	stage('Artifactory configuration') {
	   steps {
		script {
			rtMaven.tool = 'Maven-3.5.3' //Maven tool name specified in Jenkins configuration
			
			rtMaven.resolver releaseRepo:'libs-release', snapshotRepo: 'libs-snapshot', server: server //Defining where Maven Build should download its dependencies from
			
			rtMaven.deployer releaserRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local', server: server //Defining where the build artifacts should be deployed to
			
			rtMaven.deployer.artifactDeploymentPatterns.addExclude("pom.xml") //Exclude artifacts from being deployed
			
			rtMaven.deployer.deployArtifacts =false // Disable artifacts deployment during Maven run
		    
			buildInfo = Artifactory.newBuildInfo() //Publishing build-Info to artifactory
			
			buildInfo.retention maxBuilds: 10, maxDays: 7, deleteBuildArtifacts: true
			}
	    }
	}

	stage('Execute Maven') {
		steps {
		   script {
		
		rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
			}
		}
		
	}

	stage('Publish build info') {
		steps {
		  script {
		server.publishBuildInfo buildInfo
		}
		}
	}
}
}