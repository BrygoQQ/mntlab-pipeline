node {
	env.GRADLE="${tool name: '3.3', type: 'gradle'}"
	env.PATH="${env.GRADLE}/bin:${env.PATH}"
	
	stage('Preparation (Checking out)') {
		checkout([$class: 'GitSCM', branches: [[name: '*/akiryushin']], userRemoteConfigs: [[url: 'https://github.com/BrygoQQ/mntlab-pipeline.git']]])
		sh "echo $PATH"
	}
	
	stage('Building code') {
		sh "gradle build"
	}
	
	stage('Testing code') {
		parallel firstBranch: {
			stage('Unit Tests') {
				withGradle {
					sh "gradle test"
				}
			}
    	}, secondBranch: {
			stage('Jacoco Tests') {
				withGradle {
					sh "gradle jacocoTestReport"
				}
			}
		}, thirdBranch: {
			stage('Cucumber Tests') {
				withGradle {
					sh "gradle cucumber"
				}
			}
		}
	}
	
	stage('Triggering job and fetching artefact after finishing') {
		build job: 'MNTLAB-akiryushin-child1-build-job', parameters: [string(name: 'BRANCH', value: 'akiryushin')], wait: true
		step([$class: 'CopyArtifact', projectName: "MNTLAB-akiryushin-child1-build-job", filter: 'akiryushin_dsl_script.tar.gz']);
	}
	
	stage('Packaging and Publish results') {
		sh label: '', script: 'tar -xzf akiryushin_dsl_script.tar.gz jobs.groovy'
		sh label: '', script: 'tar -czf pipeline-akiryushin-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile build/libs/gradle-simple.jar'
		archiveArtifacts 'pipeline-akiryushin-${BUILD_NUMBER}.tar.gz'
	}
	
	stage('Asking for manual approval') {
		input message: 'Deployment this artefact?', ok: 'Deploy'
	}
	
	stage('Deployment') {
		sh "java -jar build/libs/gradle-simple.jar"
	}
	
	stage('Sending status') {
		echo 'SUCCESS'
	}
}
