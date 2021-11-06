// vi:set nu ai ap aw smd showmatch tabstop=4 shiftwidth=4: 

import jenkins.model.*
jenkins = Jenkins.instance

library 'jenkins-shared-library'

def sayHello(String name = 'human') {
  echo "Hello, ${name}"
}

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
      }    
    }

    stage('Parameters'){
      steps {
        script {
          properties([
            parameters([
              [$class: 'ChoiceParameter',
                  choiceType: 'PT_SINGLE_SELECT',
                  description: 'Select Platform',
                  filterLength: 1,
                  filterable: false,
                  name: 'Platform',
                  script: [
                    $class: 'GroovyScript',
                    fallbackScript: [
                      classpath: [],
                      sandbox: false,
                      script:
                        "return['could not get packages']"
                    ],
                    script: [
                      classpath: [],
                      sandbox: false,
                      script: 
                        "return['pkg_01.tar.gz,','pkg_02.tar.gz','pkg_03.tar.gz']"
                    ]
                 ]
               ]
             ]
            )
          ]
		  )
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
