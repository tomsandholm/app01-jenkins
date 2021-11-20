// vi:set nu ap aw smd showmatch tabstop=4 shiftwidth=4: 

// library 'jenkins-shared-library'

// def sayHello(String name = 'human') {
//    echo "Hello, ${name}"
// }

pipeline {
  agent any
  options {
    timestamps()
  }  

  environment {
    CAUSE = "${currentBuild.getBuildCauses()[0].shortDescription}"
    GIT_REPO_NAME = env.GIT_URL.replaceFirst(/^.*\/([^\/]+?).git$/, '$1')
    GIT_BRANCH_NAME = "${GIT_BRANCH.split('/').size() >1 ? GIT_BRANCH.split('/')[1..-1].join('/') : GIT_BRANCH}"
  }

  stages {

    stage('stage 1') {
      steps {
        sh 'echo "Step 1"'
        sh 'echo "Step 2"'
        sh 'echo "Step 3"'
      }
    }
  }


  post {
    always {
      step([$class: 'WsCleanup'])
    }
  }
}
