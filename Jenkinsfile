pipeline {
    agent any

    tools {
        maven "Maven" // Use the correct name configured in Jenkins
    }

    environment {
        MAVEN_HOME = tool 'Maven' // Update this if the name is different
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "172.31.40.209:8081"
        NEXUS_REPOSITORY = "vprofile-release"
        NEXUS_REPO_ID = "vprofile-release"
        NEXUS_CREDENTIAL_ID = "nexuslogin"
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
                scannerHome = tool 'sonarscanner4'
            }
            steps {
                withSonarQubeEnv('sonar-pro') {
                    sh '''
                        ${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile-repo \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    '''
                }
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Publish to Nexus Repository Manager') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")

                    if (filesByGlob.size() == 0) {
                        error "*** File could not be found"
                    }

                    def artifactPath = filesByGlob[0].path
                    echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version: ${ARTVERSION}"

                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: pom.groupId,
                        version: ARTVERSION,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [artifactId: pom.artifactId, classifier: '', file: artifactPath, type: pom.packaging],
                            [artifactId: pom.artifactId, classifier: '', file: 'pom.xml', type: 'pom']
                        ]
                    )
                }
            }
        }
    }
}
