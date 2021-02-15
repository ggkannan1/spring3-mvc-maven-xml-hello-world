pipeline {
    agent any
    tools {
        // Note: this should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven "Maven3"
    }
    environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "34.204.179.187:8081/"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "mvc-maven-helloworld_snt_deploy_pipeline"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
    }
    stages {
        stage("clone code") {
            steps {
                script {
                    // Let's clone the source
                    git 'https://github.com/ggkannan1/spring3-mvc-maven-xml-hello-world.git';
                }
            }
        }
        stage('Sonarqube') {
            environment {
                 scannerHome = tool 'mysonar-scanner'
            }
            steps {
                withSonarQubeEnv('mysonar') {
                sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage("Sonar Quality Gate") {
            steps {
                timeout(time: 10, unit: 'MINUTES'){
                waitForQualityGate abortPipeline: true
                }
            }
        }
        stage("mvn build") {
            steps {
                script {
				       sh "mvn package"
                       archiveArtifacts artifacts: 'target/*.war', onlyIfSuccessful: true                
        }
            }
        }
        stage("publish to nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: '${BUILD_NUMBER}',
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                        sh "curl -v  ${NEXUS_PROTOCOL}://${NEXUS_URL}/repository/${NEXUS_REPOSITORY}/com/mkyong/${pom.artifactId}/${BUILD_NUMBER}/${pom.artifactId}-${BUILD_NUMBER}.${pom.packaging} -o result.war"
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        stage('deploy'){
            steps{
                withCredentials([usernameColonPassword(credentialsId: 'tomcat-credentials', variable: 'tomcat_cred')]) {
                sh "curl -v -u ${tomcat_cred} -T result.war 'http://18.206.169.50:8080/manager/text/deploy?path=/sonarnexusdeploy&update=true'"
                } 
            }
        }
    }
    post {
        failure {
            script {
                currentBuild.result = 'FAILURE'
            }
        }

        always {
                emailext attachLog: true, attachmentsPattern: '**/target/*.war', body: '$DEFAULT_CONTENT', compressLog: true, recipientProviders: [buildUser()], replyTo: '$DEFAULT_REPLYTO', subject: '$DEFAULT_SUBJECT', to: '$DEFAULT_RECIPIENTS'
        }
    }
}
