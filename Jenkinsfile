pipeline {
    agent any

    tools {
        maven "Maven" // Use the correct name configured in Jenkins
    }

    environment {
        MAVEN_HOME = tool 'Maven' // Update this if the name is different
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "http://35.154.14.153:8081"
        NEXUS_REPOSITORY = "loginvproject"
        NEXUS_REPO_ID = "loginvproject"
        NEXUS_CREDENTIAL_ID = "nexus"
        ARTVERSION = "${BUILD_ID}"  // Use BUILD_ID for artifact versioning
    }    

    stages {
        stage('Build') {
            steps {
                // Use Maven tool defined in Jenkins
                sh "${MAVEN_HOME}/bin/mvn clean install -DskipTests"
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn test"
            }
        }

        stage('INTEGRATION TEST') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn verify -DskipUnitTests"
            }
        }

        stage('CODE ANALYSIS WITH CHECKSTYLE') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn checkstyle:checkstyle"
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
            environment {
                scannerHome = tool 'sonarscanner'
            }
            steps {
               withSonarQubeEnv('sonarscanner') {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }
               timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                } 
            }
        }

        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: ARTVERSION,
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
                    } 
		    else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }


    }


}
