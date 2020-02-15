node {
    
	stage('Preparation (Checking out)') {
		checkout([$class: 'GitSCM', branches: [[name: '*/kiryushin']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/BrygoQQ/mntlab-pipeline']]])
	}
	
	stage('Building code') {
		sh "gradle build"
	}
	
	stage('Testing code') {
		parallel 
		firstBranch: {
			stage('Unit Tests') {
				sh "gradle test"
			}
    	}, 
		secondBranch: {
			stage('Jacoco Tests') {
				sh "gradle jacocoTestReport"
			}
		},
		thirdBranch: {
			stage('Cucumber Tests') {
				sh "gradle cucumber"
			}
		}
	}
	
	stage('Triggering job and fetching artefact after finishing') {
		build job: 'MNTLAB-akiryushin-child1-build-job', parameters: [string(name: 'BRANCH', value: 'akiryushin')]
		step {
			copyArtifacts filter: 'akiryushin_dsl_script.tar.gz', projectName: 'MNTLAB-akiryushin-child1-build-job'
		}
	}
	
	stage('Packaging and Publish results') {
		sh label: '', script: 'tar -xzf akiryushin_dsl_script.tar.gz jobs.groovy'
		sh label: '', script: 'tar -czf pipeline-akiryushin-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile gradle-simple.jar'
		archiveArtifacts 'pipeline-akiryushin-${BUILD_NUMBER}.tar.gz'
	}
	
	stage('Asking for manual approval') {
		input 'Deployment this artefact?', ok: 'Deploy'
	}
	
	stage('Deployment') {
		sh label: '', script: 'java -jar gradle-simple.jar'
	}
	
	stage('Sending status') {
		echo 'SUCCESS'
	}
}
