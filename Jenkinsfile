
pipeline {
  agent {
    node {
      label 'LV2015'
    }
  }
  environment {
    LV_VER = 2015
	LV_BIT = 32
    G_CLI_PARAMS = "--lv-ver ${env.LV_VER}"
	VERSION = "0.1.0"
	FULL_VERSION = VersionNumber(versionNumberString: '${BUILDS_ALL_TIME,X}', versionPrefix: "${VERSION}." , worstResultForIncrement: 'FAILURE') 
  }
  options {
    timeout(time:45, unit: 'MINUTES')
	buildDiscarder(logRotator(numToKeepStr: '100'))
  }
  stages {
    stage('Setup') {
      steps {
	    script {
		  currentBuild.displayName = "${env.FULL_VERSION}"
		}
		echo "Building ${FULL_VERSION}"
		bat "if not exist builds mkdir builds"
        bat 'g-cli %G_CLI_PARAMS% vipcApply -- "ext\\G-UUID Dependencies.vipc" %LV_VER% %LV_BIT%'
      }
    }

    stage('Unit Test LabVIEW') {
      steps {
        bat 'g-cli %G_CLI_PARAMS% viTester --  -xml "test_results.xml" "src\\G-UUID.lvproj"'
		junit 'test_results.xml'
      }
    }
    stage('Build Outputs') {
      steps {
		bat 'g-cli %G_CLI_PARAMS% vipBuild -- -versionNumber %FULL_VERSION% "src\\G UUID.vipb"'

      }
    }
	stage('VI Analyzer') {
      steps {
        bat 'g-cli %G_CLI_PARAMS% wiresmith\\viAnalyzer --  -report "vi_analyzer.xml" "src\\G-UUID.lvproj"'
		recordIssues enabledForFailure: true, qualityGates: [[threshold: 1, type: 'TOTAL_ERROR', unstable: true]], tools: [checkStyle(pattern: 'vi_analyzer.xml')]
      }
    }
	stage('Release Steps') {
		when { buildingTag() }
		steps {
		    script {
			  currentBuild.description = "${env.VERSION} Release"
			  currentBuild.keepLog = true
			}
		}
	}
  }
  post {
    always {
      step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: emailextrecipients([culprits(), requestor(), upstreamDevelopers()]), sendToIndividuals: true])
	  archiveArtifacts artifacts: 'builds/*.zip', fingerprint: true
	  bat 'g-cli %G_CLI_PARAMS% quitLabVIEW'
    }

  }
  
}