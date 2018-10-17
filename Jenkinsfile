#!groovy
import groovy.transform.Field

@Library("liferay-sdlc-jenkins-lib") import static org.liferay.sdlc.SDLCPrUtilities.*

@Field final gitRepository = 'wiredlabs/ONECLICK2'
@Field final projectName = "ONECLICK2"
@Field final projectKey  = "ONECLICK2"

def onError() {
	handleError(gitRepository, "jhony.gomes@objective.com.br", "github_wiredlabs_ONECLICK2")
}

node ("ONECLICK2||shared-pool") {
	try {
		stage('Checkout') {
			checkout scm
		}

		stage('Setup') {
            println 'Cleaning up leaked Tomcat process...'
            sh "pkill -9 -f 'java.*tomcat.*start'  || echo 'No Tomcat process found.'"

            if (fileExists("bundles"))
                deleteRecursive: "bundles"

			prInit(projectKey, projectName);
			
			gradlew 'clean'
		}

		stage('Init Bundle') {
			gradlew 'initBundle -Pliferay.workspace.environment=ci'
		}

		stage('Build') {
			try {
				gradlew 'build -x test -x testIntegration -x functionalTest'
			} catch (exc) {
				onError()
				throw exc
			}
		}

		stage('Test') {
            try {
                wrap([$class: 'Xvnc']) {
                    gradlew 'test testIntegration killInsistenceLayer functionalTest --continue'
                }
            } catch (exc) {
                onError()
                currentBuild.result = 'UNSTABLE'
            } finally {
                junit testResults: '**/build/test-results/test/*.xml', allowEmptyResults: true
                junit testResults: '**/build/test-results/testIntegration/*.xml', allowEmptyResults: true
                junit testResults: '**/build/test-results/performTests/*.xml', allowEmptyResults: true
                archiveArtifacts artifacts: '**/build/test-results/functionalTest/attachments/**', fingerprint: true, allowEmptyArchive: true
            }
		}

		stage('Sonar') {
			try {
				sonarqube gitRepository
			} catch (exc) {
				throw exc
			} finally {
				onError()
			}
		}
	}finally {
		stage('Cleanup') {
			dir(workspace) {
				deleteDir();
			}
		}
	}
}
//foo
