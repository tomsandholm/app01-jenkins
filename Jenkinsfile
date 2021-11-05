// vi:set nu ai ap aw smd showmatch tabstop=4 shiftwidth=4: 

library 'jenkins-shared-library'

def sayHello(String name = 'human') {
  echo "Hello, ${name}"
}

pipeline {
  agent any
  options {
    timestamps();
  }

  parameters {
    string(
	  description: """ \
	    Whether to download a custom platform \
		""",
      defaultValue: 'latest',
	  name: 'PKG_FILE'
	)
  }

  environment {
    CAUSE = "${currentBuild.getBuildCauses()[0].shortDescription}"
	GIT_REPO_NAME = env.GIT_URL.replaceFirst(/^.*\/([^\/]+?).git$/, '$1')
	GIT_BRANCH_NAME = "${GIT_BRANCH.split('/').size() >1 ? GIT_BRANCH.split('/')[1..-1].join('/') : GIT_BRANCH}"
  }

  stages {

    stage('checkout') {
      steps {
		buildDescription "repo: ${env.GIT_REPO_NAME}  branch: ${env.GIT_BRANCH_NAME}"
        checkout scm
      }
    }

    stage('setup') {
      steps {
        sh """
          sudo apt-get install -y autoconf automake libtool checkinstall
          autoreconf --verbose --install --force
          ./configure
        """
      }
    }

    stage('build') {
      steps {
        sh """
          make all
          make dist
          make package
        """
      }
    }

	stage('check parent') {
	  steps {
		  echo "Build caused by ${env.CAUSE}"
		  echo 'use single quotes Build caused by ${env.CAUSE}'
		  sayHello 'Thomas'
		  helloWorld 'this is from the jenkins-shared-library'
		  echo "Value of PKG_FILE is ${PKG_FILE}"
      }	
	}

	stage('check PKG_FILE') {
	  steps {
	    script {
		  if ( "$PKG_FILE" != "latest" ) {
	  	    sh """
			  echo "the PKG_FILE is not latest"
		      echo "so get the package"
              # this is comment in NOT LATEST
		      pwd
		      ls -lia
            """
          }
	    }
      }
    }
  }
  post {
    always {
      archiveArtifacts artifacts: 'app*.deb', onlyIfSuccessful: true
      step([$class: 'WsCleanup'])
    }
  }
}
