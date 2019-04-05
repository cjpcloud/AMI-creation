branchName = "preprod"
qaEmailId ="kranthijawajiwar963@gmail.com"
repositoryName = "demo"
waitingTime = 24
pipeline { 
    agent any 
    stages {
        stage('checkout') { 
            steps { 
               git branch: 'dev', url: 'https://github.com/cdpipeline/vetinary-care-solutions.git'

            }
        }
        stage("build & SonarQube analysis") {
          steps {
              
                 withSonarQubeEnv('SonarQube') {
             
                 sh 'mvn clean package sonar:sonar'
              
              
              }
          
      }
        }
    
     stage ('junit') {
         steps {
              junit 'target/surefire-reports/*.xml'
         }
     }
     stage ('deploy s3') {
         steps {
              s3Upload consoleLogLevel: 'INFO', dontWaitForConcurrentBuildCompletion: false, entries: [[bucket: 'project544', excludedFile: '', flatten: false, gzipFiles: false, keepForever: false, managedArtifacts: false, noUploadOnFailure: false, selectedRegion: 'us-east-1', showDirectlyInBrowser: false, sourceFile: '**/*.war', storageClass: 'STANDARD', uploadFromSlave: false, useServerSideEncryption: false]], pluginFailureResultConstraint: 'SUCCESS', profileName: 's3', userMetadata: []
         }
     }
    

stage ('DEV') {
        steps {
      ansiblePlaybook becomeUser: null, credentialsId: '30c7beee-57c0-4480-aa77-4b1c1db47daf', installation: 'ansible', inventory: 'hosts', limit: 'all', playbook: 'deploy.yml', sudoUser: null
        }
    }
    }
}

              if (branchName == "preprod") {
              promoteStage()
              }

         def promoteStage(){
              // Stage: promote
                   stage ('Appprove to proceed'){	
                       notifyQA()
	               proceedConfirmation("proceed1","promote to Prod ?")
                  }
                  	node{
	           stage ('Promote artifacts to UAT'){
	               			echo 'Hi Cloudjournee'

	           }
		    }
		}

		def notifyQA(String buildStatus = 'STARTED') {
	        // build status of null means successful
	        buildStatus =  buildStatus ?: 'SUCCESSFUL'
	        def toList = qaEmailId
	        def subject = "QA: '${repositoryName}' artifact ready for promotion to Prod"
	        def summary = "${subject} (${env.BUILD_URL})"
	        def details = """
	        <p>Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' is ready to be promoted from DEV to QA.</p>
	        <p>Click here to move the library into the QA artifactory for testing. "<a href="${env.BUILD_URL}/input">${env.JOB_NAME}[${env.BUILD_NUMBER}]/</a>"</p>
	   """
	        emailext body: details,mimeType: 'text/html', subject: subject, to: toList
                }
		def proceedConfirmation(String id, String message) {	
                def userInput = true
                def didTimeout = false
                try {
                timeout(time: waitingTime, unit: 'HOURS') { //
                userInput = input(
                id: "${id}", message: "${message}", parameters: [
                [$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Confirm to proceed !']
             ])
          }
      } 
		catch(e) { // timeout reached or input false
                def user = e.getCauses()[0].getUser()
                if('SYSTEM' == user.toString()) { // SYSTEM means timeout.
                didTimeout = true
                if (didTimeout) {
                echo "no input was received abefore timeout"
                currentBuild.result = "FAILURE"
                throw e
            } 
		else if (userInput == true) {
                echo "this was successful"
            } 
		else {
                userInput = false
                echo "this was not successful"
                currentBuild.result = "FAILURE"
                println("catch exeption. currentBuild.result: ${currentBuild.result}")
                throw e
            }
       } 
		else {
                userInput = false
                echo "Aborted by: [${user}]"
     }
   }
 }
---------------------------------------------------------------------

pipeline { 
    agent any 
    stages {
        stage('checkout') { 
            steps { 
                
git branch: 'prod', credentialsId: 'f87222f2-e0e2-4681-bf46-3ac8a487bf3c', url: 'https://github.com/cdpipeline/vetinary-care-solutions.git'           
}
            }
        
       stage("build") {
          steps {
              
                sh 'mvn clean package'
              
              
              }
          
      }
      
        
     stage ('junit') {
         steps {
             script {
              junit 'target/surefire-reports/*.xml'
         }
         }
     }
     stage ('deploy s3') {
         steps {
             
              s3Upload consoleLogLevel: 'INFO', dontWaitForConcurrentBuildCompletion: false, entries: [[bucket: 'project544', excludedFile: '', flatten: false, gzipFiles: false, keepForever: false, managedArtifacts: false, noUploadOnFailure: false, selectedRegion: 'us-east-1', showDirectlyInBrowser: false, sourceFile: '**/*.war', storageClass: 'STANDARD', uploadFromSlave: false, useServerSideEncryption: false]], pluginFailureResultConstraint: 'SUCCESS', profileName: 's3', userMetadata: []
         }
     }
    

stage ('prod') {
        steps {
           script {
            ansiblePlaybook becomeUser: 'null', credentialsId: '2b7898d6-c938-4f27-9860-4f3a49f7aa3b', installation: 'ansible', inventory: 'hosts', limit: 'all', playbook: 'deploy.yml', sudoUser: 'null'
        }
        }
    
post
    {
         success
        {
            script
            {
            
		    mail to: 'kranthijawajiwar963@gmail.com',
                     subject: "Build + Condition Pass",
                     body: "Build got success check status @ ${env.BUILD_URL}"
                
            }
	}
        
          failure
	   {
		   script
		   {
                mail to: 'kranthijawajiwar963@gmail.com',
                     subject: "Build fail + Condition Pass",
                     body: "Build got success check status @ ${env.BUILD_URL}"
                 
            }
        }
    }
    
}}
}
