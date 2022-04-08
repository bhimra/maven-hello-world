pipeline {
    agent any
    tools {
        maven "maven"   
    }

    environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "192.168.231.153:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "my-local"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
    }

    stages {
         stage('checkout SCM') {
            steps {
                git 'https://github.com/bhimra/maven-hello-world.git'
           }
        }
        stage('build local repo') {
            steps {
                sh 'cp -r /var/lib/jenkins/workspace/maven-beta/repo/*.jar /root/.m2/repository/org/apache/maven/plugins/maven-deploy-plugin/2.8.2'
                sh 'mvn deploy:deploy-file -Durl=file:///root/.m2/repository/org/apache/maven/plugins/maven-deploy-plugin/2.8.2 -Dfile=/repo/sticky-deployer-embedded-0.9-sample.jar -DgroupId=net.stickycode.deploy -DartifactId=sticky-deployer-embedded -Dpackaging=jar -Dversion=0.9'
           }
        }
        
        stage("Maven Build") {
            steps {
                script {
                    sh "mvn package -DskipTests=true"
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
              
//         stage ("Deploy on target machine") {
//             steps {
//                 sh '''
//                 ssh -T docker@192.168.231.153 << ENDSSH
//                 cd /home/docker
//                 sudo mkdir active
//                 cd active/
//                 sudo wget --user=admin --password=admin  http://192.168.231.153:8081/repository/my-app/com/mycompany/app/my-app/1.0-SNAPSHOT/my-app-1.0-*.jar
//                 sudo chmod +x .
//                 sudo java -jar *.jar  
// ENDSSH
//       '''
//                 }
//             }   
    }  
}
