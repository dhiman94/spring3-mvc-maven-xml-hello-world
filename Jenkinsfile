pipeline{
    agent any
    tools {
        maven 'maven'
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "ec2-15-206-127-71.ap-south-1.compute.amazonaws.com:8081" //change this accordingly
        NEXUS_REPOSITORY = "spring-app"
        NEXUS_CREDENTIAL_ID = "nexus_credentials"
    }
    stages{
        stage("Git checkout"){
            steps{
                git changelog: false, credentialsId: 'github_credentials', poll: false, url: 'https://github.com/dhiman94/spring3-mvc-maven-xml-hello-world.git'
            }
        }
        stage("Maven build"){
            steps{
                sh label: '', script: 'mvn clean package'
            }
        }
        stage("Publish artifacts to Nexus"){
            steps{
                script{
                    //read pom.xml file
                    pom = readMavenPom file: 'pom.xml';
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // extract artifact path
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;
                    if(artifactExists){
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
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                    
                }
            }
            
        }
        
        
    }
    
}
