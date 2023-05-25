// Define COLOR_MAP to be used by Slack. 'good' is green, 'danger' is red
def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]
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
        NEXUS_PASS = 'admin'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vprofile-maven-central'
        NEXUSIP = '172.31.20.27'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vprofile-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
        NEXUSPASS = credentials('nexuspass')

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
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checksum Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }


        stage ('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE
                    // true = set pipeline to UNSTABLE
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Publish to Nexus Repository Manager") {
            steps {
            // Code reference in github.com/jenkinsci/nexus-artifact-uploader-plugin
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_LOGIN}",
                    artifacts: [
                        [artifactId: 'vproapp',
                         classifier: '',
                         file: 'target/vprofile-v2.war',
                         type: 'war']
                    ]
                 )
            }
        }

        stage{'Ansible Deploy to staging') {
            steps {
                ansiblePlaybook([
                inventory   :   'ansible/stage.inventory',
                playbook    :   'ansible/site.yml',
                installation:   'ansible',
                colorized   :   true,
                credentialsId:  'applogin',
                disableHostKeyChecking: true,
                extraVars   :   [
                        USER: "admin",
                        PASS: "${NEXUSPASS}",
                        nexusip: "172.31.20.27",
                        reponame: "vprofile-release",
                        groupid: "QA",
                        time: "${env.BUILD_TIMESTAMP}",
                        build: "${env.BUILD_ID}",
                        artifactid: "vproapp",
                        vprofile_version: "vproapp-${env.BUILD_ID}-${env.BUILD_TIMESTAMP}.war"
                    ]
                ])
            }
        }

    }
// Post stage execution, will run regardless of build status
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkins-ci',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}. \n More info at: ${env.BUILD_URL}"
        }
    }

}