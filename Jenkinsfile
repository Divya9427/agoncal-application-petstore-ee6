pipeline {
	    agent any
	    
	    
	    environment {
	        NEXUS_VERSION = "nexus3"
	        NEXUS_PROTOCOL = "http"
	        NEXUS_URL = "34.69.119.221:8081"     
	        NEXUS_REPOSITORY = "Jwalarepositoy"
	        NEXUS_CREDENTIAL_ID = "nexuscredentials"
	    }
	
	    stages {
	        stage ('Compile') {
	
	            steps {
	                withMaven(maven : 'maven_3_5_0') {
	                    sh 'mvn clean package'
	                }
	            }
	        }
	        stage ('Testing Stage') {
	
	            steps {
	                withMaven(maven : 'maven_3_5_0') {
	                    sh 'mvn test'
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
	}
