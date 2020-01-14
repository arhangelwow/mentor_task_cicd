#!groovy
//Scripted Pipeline
node ('Slave1Centos') {//node

def mvnHome = tool 'MavenPet_3.6.3'  //global variable for maven installations in jenkins
        timestamps{//timestamps, Prepend all console output generated by the Pipeline run with the time at which the line was emitted
            def IMAGE = readMavenPom().getArtifactId()
            def VERSION = readMavenPom().getVersion()

        stage ('GitHub Preparation') {//stage 1
            git 'https://github.com/arhangelwow/spring-petclinic'
        }//stage 1

        stage ('Maven clean + compile') {//stage 2
            withEnv(["PATH+MAVEN=${mvnHome}/bin"]) {//withEnv
                sh "mvn clean compile"
            }//withEnv
        }//stage 2

        stage ('Maven package') {//stage 3
            withEnv(["PATH+MAVEN=${mvnHome}/bin"]) {//withEnv
                sh "mvn package"
            }//withEnv
      }//stage 3
      
             stage('SonarQube Analysis') {//stage 4
           withSonarQubeEnv('sonar') {
               withEnv(["PATH+MAVEN=${mvnHome}/bin"]) {//withEnv
                 sh '''cd /var/lib/jenkins/workspace/spring_petclinic_pipeline/; 
                     mvn sonar:sonar'''
               }//withEnv
          }
   }//stage 4

      stage ('Artifactory') {//stage 5
        withEnv(["PATH+MAVEN=${mvnHome}/bin"]) {//withEnv
            def server = Artifactory.server 'Art'
            def uploadSpec = 
            """
            {
            "files": [
                {
                    "pattern": "${env.WORKSPACE}/target/*.jar",
                    "target": "libs-release-local/${IMAGE}_${VERSION}_${BUILD_TIMESTAMP}_${BUILD_NUMBER}.jar",
                    "props": "type=jar;status=ready"
                }
              ]
            }
            """
            buildInfo=server.upload(uploadSpec) 
            server.publishBuildInfo(buildInfo) 
            buildName '${JOB_NAME}_${BUILD_NUMBER}'
        }//withEnv
    }//stage 5

    stage('Start Petclinic Website, Deploy Stage') {//stage 6
            withEnv(["PATH+MAVEN=${mvnHome}/bin", "JENKINS_NODE_COOKIE=do_not_kill", "BUILD_ID=do_not_kill"]) {//withEnv
                    def server = Artifactory.server 'Art'
                    def downloadSpec = 
                    """{
                     "files": [
                      {
                    "pattern": "libs-release-local/*_${BUILD_NUMBER}.jar",
                    "target": "${WORKSPACE}/builds/",
                    "props": "type=jar;status=ready"
                        }
                     ]
                    }
                    """
                    buildInfo=server.download(downloadSpec) 
                    
                sh """cd ${WORKSPACE}/
                mvn clean install
                cd ${WORKSPACE}/builds/
                nohup java -jar ${IMAGE}_${VERSION}_${BUILD_TIMESTAMP}_${BUILD_NUMBER}.jar &"""
            }//withEnv
    }//stage 6
    
   } //timestamps  
 }//node 
