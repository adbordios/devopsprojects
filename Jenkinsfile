pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }
    environment {
        // Define variables on settings.xml file, refer to Nexus repo names and Jenkins credential name
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'adminpassword'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vprofile-maven-central'
        NEXUSIP = '172.31.23.72'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vprofile-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
    }
    stages {
        stage ('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install' // Pass settings file and skip unit tests
            }
            post {
                success {
                    echo "Now Archiving."
                    // archiveArtifacts plugin installed in Jenkins, get ll files with .war extension
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage ('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Checksum Analysis') {
            steps {
                sh mvn checkstyle:checkstyle
            }
        }
    }
}