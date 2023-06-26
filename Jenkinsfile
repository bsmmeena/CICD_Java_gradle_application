pipeline{
    agent any
    environment {
        VERSION = "${env.BUILD_ID}"
    }
    tools{
        maven 'Maven'
    }
    stages{
        stage('sonar quality check'){
            steps{
            script{
                withSonarQubeEnv(credentialsId: 'sonar-token') {
                sh 'mvn clean package sonar:sonar'    
                }

            }
          }
        }
        stage('sonar quality gate status'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('docker build & docker push to Nexus Server'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_cred')]) {
                        sh '''
                        docker build -t 142.44.249.127:8083/springapp:${VERSION} .
                        docker login -u admin -p $nexus_cred 142.44.249.127:8083
                        docker push 142.44.249.127:8083/springapp:${VERSION}
                        docker rmi 142.44.249.127:8083/springapp:${VERSION}
                    '''
                    }
                    
                }
            }
        }
        stage('Identifing misconfigs using datree in helm charts'){
            steps{
                script{
                    dir('kubernetes/myapp/') { 
                        withEnv(['DATREE_TOKEN=f8b10f3b-9a2f-460d-ad09-4617b7921cae']) {
                            sh 'helm datree test .'
                        }
                    }
                }
            }
        }
        stage('Pushing the helm chart to nuxes repo'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_cred')]) {
                        dir('kubernetes/') { 
                            sh '''
                            helmversion=$(helm show chart myapp | grep version | cut -f2 -d: | tr -d ' ')
                            tar -czvf myapp-${helmversion}.tgz myapp/
                            curl -u admin:$nexus_cred http://142.44.249.127:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                        }
                    }
                }
            }
        }
          stage('Deploying application on k8s cluster') {
            steps {
               script{
                   withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_cred')]) {
                        dir('kubernetes/') {
                          sh 'helm upgrade --install --set image.repository="142.44.249.127:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
                        
                    }
                   }
               }
            }
          }
                /*stage ('Jfrog Artifacory'){
	steps{
	rtServer (
	id: "Artifactory",
	url: 'http://209.126.4.53:8082/artifactory',
	username: 'rawat',
	password: 'Rawat@#23',
	bypassProxy: true,
	timeout: 300

    )

	}
}

stage ('Upload to Jfrog Server'){
	steps{
	rtUpload (
	serverId:"Artifactory",
	spec: '''{
	"files": [
	{
	"pattern": "*.jar",
	"target": "logic-ops-lab-libs-snapshot-local"

	}
	]
	}''',
	)
	}
}

stage ('Jfrog publish and build info'){
	steps{
	rtPublishBuildInfo(
	serverId:"Artifactory"
	)
	}

}*/
    }
   //post {
     //   always {
       //     emailext attachLog: true, attachmentsPattern: '**/*.text',
         //       body: "<br> Job Status = ${currentBuild.currentResult} <br> Job Name = ${env.JOB_NAME} <br> Build Number =  ${env.BUILD_NUMBER}\n <br> For Job More info visit here: ${env.BUILD_URL}",
           //     recipientProviders: [developers(), requestor()],
             //   subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}", to: "meenabs14@gmail.com"
            
        //}
    //}
}
