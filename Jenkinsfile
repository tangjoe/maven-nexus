pipeline {
    agent any

    parameters {
        string(name:'repoUrl', defaultValue:'https://github.com/tangjoe/spring-boot.git', description:'代码路径')
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "nexus:8081"
        NEXUS_REPOSITORY = "maven-private-repo"
        NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
    }

    tools {
        maven 'maven3.6'
        jdk   'jdk8'
    }

    stages {
        stage('Checkout source') {
            steps {
                echo "//Stage-1 === Checkout source ==="
                git url:params.repoUrl
            }
        }
        stage('Code Qualify Check via SonarQube') {
            steps {
                echo "//Stage-2 === Code Quality Check via SonarQube  ==="
                script {
                    def scannerHome = tool 'SonarQube Scanner';
                        withSonarQubeEnv("SonarQube") {
                            sh "${tool("SonarQube Scanner")}/bin/sonar-scanner \
                                -Dsonar.projectKey=hello-sb \
                                -Dsonar.sources=. \
                                -Dsonar.css.node=. \
                                -Dsonar.host.url=http://sonarqube:9000 \
                                -Dsonar.login=9e90f99c041ef33609b50f60d205c6261e8e7ff5"
                        }
                }
            }
        }
        stage('Compile') {
            steps {
                echo "//Stage-3 === complile project ==="
                sh 'mvn clean test'
            }
        }
        stage('Build Fat Jars') {
            steps {
                echo "//Stage-4 === build jars ==="
                sh 'mvn package'
            }   
        }   
        stage('Publish to Nexus Repository Manager') {
            steps {
                echo "//Stage-5 === publish to nexus ==="
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}";
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if (artifactExists) {
                        echo "*** File:${artifactPath},group:${pom.groupId},packaging:${pom.packaging},version:${pom.version}";

                        nexusArtifactUpLoader (
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
    }
}
