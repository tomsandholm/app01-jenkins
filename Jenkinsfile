// vi:set nu ap aw smd showmatch tabstop=4 shiftwidth=4: 

// library 'jenkins-shared-library'

// def sayHello(String name = 'human') {
//   echo "Hello, ${name}"
// }

def choice = "['latest', 'pkg_01','pkg_02','pkg_03','pkg_04']"

pipeline {
  agent any
  options {
    timestamps();
  }

  environment {
    choice = "['latest', 'pkg_01','pkg_02','pkg_03','pkg_04']"
    CAUSE = "${currentBuild.getBuildCauses()[0].shortDescription}"
    GIT_REPO_NAME = env.GIT_URL.replaceFirst(/^.*\/([^\/]+?).git$/, '$1')
    GIT_BRANCH_NAME = "${GIT_BRANCH.split('/').size() >1 ? GIT_BRANCH.split('/')[1..-1].join('/') : GIT_BRANCH}"
  }

  stages {

    stage('forms') {
      steps {
        script {
          properties ([
            parameters([
              [$class: 'ChoiceParameter',
                  choiceType: 'PT_SINGLE_SELECT',
                  description: 'Specify Platform Version',
                  filterLength: 1,
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
                      script:
                        "return ['latest', 'pkg_01','pkg_02','pkg_03','pkg_04']"
                    ]
                  ]
              ]
            ])
          ])
        }
      }
    }

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
