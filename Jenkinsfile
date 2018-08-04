#!groovy​
def getLabel() {
    // Which server to run this on.
    return "master"
}
// Choose the site name based on git name and if it is a Pull Request or branch.
def getSitename() {
    SITENAME = "example"
    // env.GIT_URL[env.GIT_URL.lastIndexOf('/')+1..-5]
    if (env.CHANGE_BRANCH && !env.CHANGE_FORK){
             return "${SITENAME}-${env.CHANGE_BRANCH.toLowerCase()}"
         }
         else{
             return "${SITENAME}-${env.BRANCH_NAME.toLowerCase()}"
         }
}
pipeline {
  environment {
    // Database backup name
    X_DB_BACKUP = "example.sql.gz"
    // Database User to use on the testing server
    X_DB_USER = "example"
    // The live URL, used for stage file proxy and WP find-replace
    X_LIVE_DOMAIN = "https://www.example.com"
    // Slack Channel
    X_SLACK_CHANNEL = "#example-deploys"
    // Code paths for phpcs checks, space delimited
    X_CODE = "web/sites/all/modules/custom/ web/sites/all/themes/allstrap/"
    // Code paths for phpcs ignore, comma delimited
    X_IGNORE = "*/vendor_reports/includes/*,*/qunit/*"
  }
  agent {
    node {
      label "${getLabel()}"
    }
  }

  options {
    // do not run more than one build per branch
      disableConcurrentBuilds()
    // lock based on branch name, pull requests use branch name
      lock(resource: "example", inversePrecedence: true)
    // timeout after 1 hours
      timeout(time: 1, unit: 'HOURS')
    // keep 7 jobs
      buildDiscarder(logRotator(numToKeepStr: '7'))
    // mark output with times
      timestamps()
  }
  stages {
    stage("Checkout") {
      steps {
        checkout scm
      }
    }
    stage("Setup") {
      when {
        // Only build if the site is new
        expression {
          return !fileExists("./web/.htaccess")
        }
      }
      steps {
        echo "Build variables"
        echo "${getSitename()}"
        echo "${getLabel()}"
        echo "Build variables end"
        echo "Build Drush Start"
      }
    }
    stage("Tests") {
      when {
        expression { env.CHANGE_TARGET == 'master' }
      }
        failFast true
        parallel {
            stage('Behat') {
              steps {
                echo "Run Beaht Tests"
              }
            }
            stage('PHPcs') {
              steps {
                echo "Run PHPcs Tests"
              }
            }
            stage('phpUnit') {
              steps {
                echo "Run phpUnit Tests"
              }
            }
        }
    }
    stage ('Deploy Code') {
        when {
            branch 'master'
        }
        steps {
          echo "Deply Code"
        }
    }
  }
  post {
    success {
          echo "success!"
    }
    failure {
          echo "failure!"
    }
    unstable {
          echo "unstable."
    }
  }
}
