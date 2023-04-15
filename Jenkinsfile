pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }
    environment {
        // Define variables on settings.xml file, refer to Nexus repo names and Jenkins credential name
        SNAP-REPO = 'vprofile-snapshot'
        NEXUS-USER = 'admin'
        NEXUS-PASS = 'adminpassword'
        RELEASE-REPO = 'vprofile-release'
        CENTRAL-REPO = 'vprofile-maven-central'
        NEXUSIP = '172.31.23.72'
        NEXUSPORT = '8081'
        NEXUS_GRP-REPO = 'vprofile-maven-group'
        NEXUS-LOGIN = 'nexuslogin'
    }
    stages {
        stage ('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install' // Pass settings file and skip unit tests
            }
        }
    }
}