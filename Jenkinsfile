// vi:set nu ai ap aw smd showmatch tabstop=4 shiftwidth=4: 

library 'jenkins-shared-library'

def sayHello(String name = 'human') {
  echo "Hello, ${name}"
}

def getChoices() {
  sh """
    out=`ls /var/lib/jenkins/packages/pkg_* | xargs -n1 basename | cut -d'.' -f1 | sed "s/^/\'/g" | sed "s/$/\'/g"`
	new=`echo $out | sed "s/ /,/g"`
	pkgs=`echo "[ ${new} ]" `
	echo $pkgs
  """
}

pipeline {
  agent any
  options {
    timestamps();
  }

  choices = sh (
    script: "ls /var/lib/jenkins/packages/pkg_* | xargs -n1 basename | cut -d'.' -f1",
	returnStatus: true
  ) == 0

  parameters {
	choice (
	  name: 'Platforms',
	  choices: [ "$choices" ],
	  description: 'Select the Platform package'
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
