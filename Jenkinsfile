pipeline {
    agent any
    
    
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "34.66.64.75:8081"
        NEXUS_REPOSITORY = "samplesnapshot"
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
    }

    stages {
        stage ('Compile') {

            steps {
                withMaven(maven : 'maven_3_5_0') {
                    sh 'mvn clean compile'
                }
            }
        }
        
           stage('SonarQube analysis') {
      steps {
           withMaven(maven : 'maven_3_5_0') {
        withSonarQubeEnv('sonar') {
          sh 'mvn clean package sonar:sonar'
        }
      }
    }
           }

        stage("Quality Gate"){
             steps {
         timeout(time: 5, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
      }  
        }
        stage ('Testing Stage') {

            steps {
                withMaven(maven : 'maven_3_5_0') {
                    sh 'mvn clean test'
                }
            }
        }
         stage("publish to nexus") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],    
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }

    }
    //post {
       //always {
            //archiveArtifacts artifacts: 'build/libs/**/*.jar', fingerprint: true
           //junit 'target/surefire-reports/*.xml'
           //junit '*.xml'
          // jacoco(
         //      execPattern: 'target/*.exec',
      //classPattern: 'target/classes',
      //sourcePattern: 'src/main/java',
      //exclusionPattern: 'src/test*'
      //-DmaximumBranchCoverage: '70',
      //-DmaximumClassCoverage: '70',
      //-DmaximumComplexityCoverage: '70',
      //-DmaximumInstructionCoverage: '100',
      //-DmaximumLineCoverage: '70',
      //-DmaximumMethodCoverage: '70',
      //-DrunAlways: true
     //      )
    //    }
   //} 
    post {
        always {
          junit "**/build/test-results/*.xml"
          step([
              $class         : 'FindBugsPublisher',
              pattern        : 'build/reports/findbugs/*.xml',
              canRunOnFailed : true
          ])
          step([
              $class         : 'PmdPublisher',
              pattern        : 'build/reports/pmd/*.xml',
              canRunOnFailed : true
          ])
          step([
              $class           : 'JacocoPublisher',
              execPattern      : 'build/jacoco/jacoco.exec',
              classPattern     : 'build/classes/main',
              sourcePattern    : 'src/main/java',
              exclusionPattern : '**/*Test.class'
          ])
          publishHTML([
              allowMissing          : false,
              alwaysLinkToLastBuild : false,
              keepAll               : true,
              reportDir             : 'build/asciidoc/html5',
              reportFiles           : 'index.html',
              reportTitles          : "API Documentation",
              reportName            : "API Documentation"
          ])
         
        }
      }
}
