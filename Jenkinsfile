#!/usr/bin/env groovy

pipeline {
  agent { label 'executor-v2' }

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '30'))
  }

  stages {
    stage('Test 2.3') {
      environment {
        RUBY_VERSION = '2.3.5'
      }
      steps {
        sh './test.sh'
        junit 'spec/reports/*.xml'
      }
    }

    stage('Test 2.4') {
      environment {
        RUBY_VERSION = '2.4.2'
      }
      steps {
        sh './test.sh'
        junit 'spec/reports/*.xml'
      }
    }

    stage('Test 2.5') {
      environment {
        RUBY_VERSION = '2.5.1'
      }
      steps {
        sh './test.sh'
        junit 'spec/reports/*.xml'
      }
    }
    
    // Only publish to RubyGems if branch is 'master'
    // AND someone confirms this stage within 5 minutes
    stage('Publish to RubyGems?') {
      agent { label 'releaser-v2' }

      when {
        allOf {
          branch 'master'
          expression {
            boolean publish = false

            if (env.PUBLISH_GEM == "true") {
                return true
            }

            try {
              timeout(time: 5, unit: 'MINUTES') {
                input(message: 'Publish to RubyGems?')
                publish = true
              }
            } catch (final ignore) {
              publish = false
            }

            return publish
          }
        }
      }
      steps {
        // Clean up first
        sh 'docker run -i --rm -v $PWD:/src -w /src alpine/git clean -fxd'

        sh './publish.sh'

        // Clean up again...
        sh 'docker run -i --rm -v $PWD:/src -w /src alpine/git clean -fxd'
        deleteDir()
      }
    }    
  }
  
  post {
    always {
      cleanupAndNotify(currentBuild.currentResult)
    }
  }
}
