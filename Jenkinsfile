// vi:set nu ai ap aw smd showmatch tabstop=4 shiftwidth=4: 

// library 'jenkins-shared-library'

// def sayHello(String name = 'human') {
//   echo "Hello, ${name}"
// }

properties([
  parameters([
    [
	  $class: 'ChoiceParameter',
	  choiceType: 'PT_SINGLE_SELECT',
	  description: '',
	  filterable: false,
	  name: 'Platform',
	  script: [
	    $class: 'GroovyScript',
		fallbackScript: [
		  classpath: [],
		  sandbox: false,
		  script:
		    "return['could not get list']"
        ],
		script: [
		  classpath: [],
		  sandbox: false,
		  script: '''
		    def sout = new StringBuffer()
		    def serr = new StringBuffer()
		    def proc = 'ls /var/lib/jenkins/package/pkg_*'.execute()
            proc.consumeProcessOutput(sout,serr)
            proc.waitForOrKill(10000)
            return[sout.tokenize()]
          '''
        ]
      ]
    ]
  ])
])

pipeline {
  agent any
  options {
    timestamps();
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
		  echo "Platform selection ${params.Platform}"
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
