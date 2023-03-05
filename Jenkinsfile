pipeline {
    agent {label 'mvn'}
    triggers { cron('0 * * * *')}
    options { buildDiscarder(logRotator(numToKeepStr: '10'))}
    environment {
        REPO_URL="sai3369"
        dockerhub=credentials('Docker')} /dockerhub reponame/
    stages {
        stage ('vcs') {
            steps{
            git branch : 'main', 
            url: 'https://github.com/rohi3369/spring-petclinic.git'
            }
        }
        stage ('Artifactory config') {
           steps{
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "Jfrogartifact",  
                    releaseRepo: 'testing-libs-release-local',
                    snapshotRepo: 'testing-libs-snapshot-local' 
                    )
            }
        }
        stage ('Exec Maven') {
             steps {
                rtMavenRun (
                    tool: "MAVEN_TOOL", 
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER"
                )
             }
         }
        stage ('Publish Build Info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "Jfrogartifact"
                )
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "mvn package sonar:sonar"
                }
            }
        }
        stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
         stage('docker image build'){
            steps{
                sh """
                docker image build -t spc:$env.BUILD_ID .
                docker tag spc:1.0 soma3369.jfrog.io/spcimage/spc:$env.BUILD_ID
                docker push soma3369.jfrog.io/spcimage/spc:$env.BUILD_ID """  
            }
        }
    } 
    // post {
    //    success{
    //         mail subject: 'Build success', 
    //              body: 'Build success', 
    //              to: 'sairohith77@gmail.com'
    //             }
    //     failure {
    //         mail subject: 'Build Failed', 
    //              body: 'Build Failed', 
    //              to: 'sairohith77@gmail.com'
        
    //             }    
    //     }   
} 
