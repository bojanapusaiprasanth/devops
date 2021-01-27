pipeline {
    agent any
    tools {
      maven 'My Maven'
      jdk 'My Java'
    }
    stages {
      stage('Git Repo details') {
        steps {
          git credentialsId: '9e853457-efea-4b6d-af63-e44e63f3c7c0', url: 'https://github.com/bojanapusaiprasanth/devops.git'
        }
      }
      stage('Compile the Code') {
        steps {
          sh 'mvn compile'
        }
      }
      stage('Code Review') {
        steps {
          sh 'mvn -P metrics pmd:pmd'
        }
        post {
          always {
            recordIssues(tools: [acuCobol(pattern: '**/target/pmd.xml', reportEncoding: 'UTF-8')])
          }
        }
      }
      stage('Junit test') {
        steps {
          sh 'mvn test'
        }
        post {
          always {
            junit '**/target/surefire-reports/*.xml'
          }
        }
      }
      stage('Metrics Check') {
        steps {
          sh 'mvn cobertura:cobertura -Dcobertura.report.format=xml'
        }
        post {
          always {
            cobertura autoUpdateHealth: false, autoUpdateStability: false, coberturaReportFile: '**/target/site/cobertura/coverage.xml', conditionalCoverageTargets: '70, 0, 0', failUnhealthy: false, failUnstable: false, lineCoverageTargets: '80, 0, 0', maxNumberOfBuilds: 0, methodCoverageTargets: '80, 0, 0', onlyStable: false, sourceEncoding: 'ASCII', zoomCoverageChart: false
          }
        }
      }
      stage('Build Package') {
        steps {
          sh 'mvn package'
        }
      }
      stage('Deploy package') {
        input {
          message "Please select YES or NO to proceed with Deployment"
          ok "Yes, We can Proceed"
          submitter "SaiPrasanth"
          parameters {
            string(name: 'PERSON', defaultValue: 'SaiPrasanth', description: 'Please provide name of person who is approving')
          }
        }
        steps {
          deploy adapters: [tomcat8(credentialsId: '13d9979a-9a5d-45d5-8258-9d8d1cf8f7b2', path: '', url: 'http://tomcat.sidhuco.in/')], contextPath: null, onFailure: false, war: '**/*.war'
        }
      }
    }
}

