pipeline {
    agent any
     tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "Maven3"
    }
    stages {
        stage('Build') {
            steps {
                // Get some code from a GitHub repository
                git credentialsId: 'github-credentials', url: 'https://github.com/ggkannan1/spring3-mvc-maven-xml-hello-world.git'
            }
        }
        stage('Package') {
            steps {
                //maven package
                sh 'mvn package'
            }
        }
        stage('Artifact') {
            steps {
		//Showing Artifact
                archiveArtifacts artifacts: 'target/*.war', followSymlinks: false
            }
        }
        stage('Deploy') {
            steps {
		//Deploy to tomcat in a static way
                withCredentials([usernameColonPassword(credentialsId: 'tomcat-credentials', variable: 'tomcat_cred')]) {
                sh "curl -v -u ${tomcat_cred} -T /var/lib/jenkins/workspace/tomcat_deploy_pipeline/target/spring3-mvc-maven-xml-hello-world-1.0-SNAPSHOT.war 'http://54.234.159.204:8080/manager/text/deploy?path=/pipeline_spring3&update=true'"
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
            step([$class: 'Mailer',
                notifyEveryUnstableBuild: true,
                recipients: "ggkannan1@gmail.com",
                sendToIndividuals: true])
        }
    }
}
