node('ILSIEDISON') {
    timestamps {
        step([$class: 'WsCleanup'])
        sh "ls -lart"
        stage ('Git Checkout') { scm() }
        stage ('Build') { build() }
        stage ('UnitTest') { unitTest() }
        stage ('Upload Artifact 2 Nexus') { uploadToNexus() }
        stage ( 'Deploy 2 App-Server-I') { deployToNginxAppServer1() }
        stage ( 'Deploy 2 App-Server-II') { deployToNginxAppServer2() }
        step([$class: 'WsCleanup'])
    }
}

// Clean the work space and Checkout the source code from Bitbucket
def scm() {

	checkout([$class: 'GitSCM', branches: [[name: 'cicd']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'd8a89bf0-2e87-4ab1-baae-6d3e476bceb3', url: 'https://rajendraraj@bitbucket.org/is_corp/reception_desk_admin_app.git']]])

}

// Verify the source code, download and install the dependencies to run and test the application.
def build() {
	
	sh 'echo jenkins | sudo -S npm install'
	sh 'cd $WORKSPACE/node_modules/ && echo jenkins | sudo -S npm install supertest mocha chai express mocha-junit-reporter'

}

// Run the UnitTest Cases.
def unitTest() {

	sh 'cd $WORKSPACE/test/ && echo jenkins | sudo -S $WORKSPACE/node_modules/mocha/bin/mocha unitTest.js --timeout 15000 --reporter mocha-junit-reporter'
	junit '**/test/*.xml'
}

// Push the binery to the Nexus Repo.
def uploadToNexus() {
	
	sh "sed 's/true/false/g' $WORKSPACE/package.json > $WORKSPACE/package1.json && rm -rf $WORKSPACE/package.json && mv $WORKSPACE/package1.json $WORKSPACE/package.json"
	sh "cd $WORKSPACE && npm publish --registry http://10.12.40.115:8082/nexus/repository/npm-internal/"

}
// Prepare the environment using Puppet and Download the binary from Nexus repo and deploy on nginc-app-server1.infostretch.com.
def deployToNginxAppServer1() {
	
	node ('nginx-app-server1') {

		sh "echo jenkins | sudo -S puppet agent -t"
		sh "echo jenkins | sudo -S /var/lib/jenkins/app/deploy.sh"
		step([$class: 'WsCleanup'])


	}

}

// Prepare the environment using Puppet and Download the binary from Nexus repo and deploy on nginc-app-serve-2.infostretch.com.
def deployToNginxAppServer2() {


	node ('nginx-app-server-2') {

		sh "echo jenkins | sudo -S puppet agent -t"
		sh "echo jenkins | sudo -S /var/lib/jenkins/app/deploy.sh"
		step([$class: 'WsCleanup'])


	}

}
