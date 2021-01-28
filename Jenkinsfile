pipeline {
    agent any
    tools {
      maven 'My maven'
      jdk 'My Java'
    }
    stages {
      stage('Git SCM') {
        steps {
          git credentialsId: 'Gitcredentials', url: 'https://github.com/bojanapusaiprasanth/devops.git'
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'Gitcredentials', url: 'https://github.com/bojanapusaiprasanth/devops.git']]])
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
            cobertura autoUpdateHealth: false,
             autoUpdateStability: false,
              coberturaReportFile: '**/target/site/cobertura/coverage.xml',
               conditionalCoverageTargets: '70, 0, 0',
                failUnhealthy: false,
                 failUnstable: false,
                  lineCoverageTargets: '80, 0, 0',
                   maxNumberOfBuilds: 0,
                    methodCoverageTargets: '80, 0, 0',
                     onlyStable: false,
                      sourceEncoding: 'ASCII',
                       zoomCoverageChart: false
          }
        }
      }
      stage('SonarQube Analysis') {
        steps {
          withSonarQubeEnv('mysonarqube') {
            sh 'mvn sonar:sonar'
          }
        }
      }
      stage('Build Package') {
        steps {
          sh 'mvn package'
        }
      }
      stage('Push Artifact To NEXUS') {
        steps {
          script {
            def mavenPom = readMavenPom file: 'pom.xml'
            nexusArtifactUploader artifacts: [
                [artifactId: 'addressbook',
                 classifier: '',
                  file: 'target/addressbook.war',
                   type: 'war'
                   ]
           ],
            credentialsId: 'Nexus',
             groupId: 'com.edurekademo.tutorial',
              nexusUrl: 'nexus.sidhuco.in',
               nexusVersion: 'nexus3',
                protocol: 'http',
                 repository: 'addressbook',
                  version: "${mavenPom.version}"
          }
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
          deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://tomcat.sidhuco.in/')], contextPath: null, onFailure: false, war: '**/*.war'
        }
      }
    }
}

