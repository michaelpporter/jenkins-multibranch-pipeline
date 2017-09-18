#!groovy​
import hudson.model.*
// Pulls Environment variables from Jenkins
import hudson.EnvVars
// used to parse git messages and send to JIRA
import groovy.json.JsonSlurperClassic
import groovy.json.JsonBuilder
import groovy.json.JsonOutput
// Set properties of the job that gets created.
properties(
    [buildDiscarder(
        logRotator(
            artifactDaysToKeepStr: '',
            artifactNumToKeepStr: '',
            // Cleans up old jobs so Jenkins does not have to work so hard.
            daysToKeepStr: '7',
            numToKeepStr: '5'
        )
     ),
    // [$class: 'RebuildSettings',
    //     autoRebuild: false,
    //     rebuildDisabled: false],
     pipelineTriggers([])])
// Variables used in this script.
    // set to /www if the repo root is the site root
    def siteFolder = '/'
    def dbUser = 'example'
    def dbPass = 'Would Be better to set this as an Environment Variable in Jenkins'
    def branchClean = "${env.BRANCH_NAME}"
    // If the branch is a feature branch make a clean URL name.
    branchClean = branchClean.replaceAll('/','-')
    // Convert to lower case for URL use.
    def branchLower = branchClean.toLowerCase()
    // Repo name and safe branch name
    def logName = "example-${branchClean}"
    def siteName = "${dbUser}-${branchLower}"
    def dbBackup = 'backup.sql.gz'
    def domainName = 'https://www.example.com'
    def dbName = siteName.replaceAll('-','_')

    // Use Milestone and lock plug-ins.
    // https://jenkins.io/blog/2016/10/16/stage-lock-milestone/
    milestone 1
    lock(resource: 'example', inversePrecedence: true) {
     node {
        stage('Build') {
          dir ("/var/jenkins_home/web-root/${siteName}") {
            checkout scm
          }
          if (!fileExists("/var/jenkins_home/web-root/${siteName}/web/sites/default/settings.local.php")) {
            sh """
            echo \"#!/bin/bash\" > /var/jenkins_home/cleanup/${logName}.txt
            echo \"# ${env.BRANCH_NAME}\" >> /var/jenkins_home/cleanup/${logName}.txt
            echo \"cd /var/jenkins_home/web-root/${siteName}/www\" >> /var/jenkins_home/cleanup/${logName}.txt
            echo \"sudo fdperms\" >> /var/jenkins_home/cleanup/${logName}.txt
            echo \"drush sql-query 'DROP DATABASE IF EXISTS ${dbName};'\" >> /var/jenkins_home/cleanup/${logName}.txt
            echo \"cd /var/jenkins_home/web-root/\" >> /var/jenkins_home/cleanup/${logName}.txt
            echo \"rm -rf /var/jenkins_home/web-root/${siteName}\" >> /var/jenkins_home/cleanup/${logName}.txt
            echo \"rm -rf /var/jenkins_home/web-root/${siteName}@tmp\" >> /var/jenkins_home/cleanup/${logName}.txt
            echo \"echo cleanup complete\" >> /var/jenkins_home/cleanup/${logName}.txt
            chmod +x /var/jenkins_home/cleanup/${logName}.txt
            """
            dir ("/var/jenkins_home/web-root/${siteName}/www") {
              // Create settings.local.php
              // Use External Build
              timeout(time:60, unit:'MINUTES') {
              // Sample not shared.
              // build job: 'global-builds/drupal-build-site',
              //   parameters: [
              //       string(name: 'DATABASE_USER', value: "${dbUser}"),
              //       string(name: 'DATABASE_NAME', value: "${dbName}"),
              //       string(name: 'DB_BACKUP', value: "${dbBackup}"),
              //       string(name: 'BRANCH_BUILD', value: "${env.BRANCH_NAME}"),
              //       string(name: 'SITE_FOLDER', value: "${siteFolder}"),
              //       string(name: 'DOMAIN_NAME', value: "${domainName}"),
              //       string(name: 'SITE_NAME', value: "${siteName}")]
              }
            }
         } // end if fileexists
    } // stage build
    } // node build
    } // end lock

    milestone 2
    lock(resource: 'example', inversePrecedence: true) {
      node {
       stage('Config') {
            dir ("/var/jenkins_home/web-root/${siteName}") {
            sh """
            # checkout scm checks out the commit hash
            git checkout ${env.BRANCH_NAME}
            git remote update origin --prune
            composer install
            cd web
            mkdir -p sites/default/files/private
            git config core.fileMode false
#            drush cim -y
#            drush updb -y
#            drush cr
            """
            // gitMessage = sh(returnStdout: true, script: 'git log -1').trim()
            // gitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
            // def triads = gitMessage.find("((?<!([A-Za-z]{1,10})-?)[A-Z]+-\\d+)")
            // // Send a message to your ticketing tool or slack
            // echo "${triads}"
            }
      } // end stage Config
      } // end node
    } // end lock

    milestone 3
    lock(resource: 'example', inversePrecedence: true) {
      node {
      stage('Test') {
      currentBuild.result = 'SUCCESS'
      def steps = [:]
      steps["behat"] = {
        // run any testing you need.
      }
      parallel steps
      } // end stage
      } // node config and test
    } // lock execadv

    milestone 4
    stage('Deploy') {
     if ("${env.BRANCH_NAME}" == 'master') {
        if (currentBuild.result.equals("SUCCESS")) {
          // Alert that the site is ready for visual test and deploy
        }
        else {
          // Alert that the site failed tests.
        }
     }
     else {
      if (currentBuild.result.equals("SUCCESS")) {
          // Alert that the site is ready on the testing server
        }
        else {
          // Alert that the site failed tests.
       }
     }
   }
