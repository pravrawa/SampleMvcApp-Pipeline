node('TAW005')
{
 try 
 { 
	stage 'Checkout'
	
			bat 'git init &&  git config http.sslVerify false'
			checkout([$class: 'GitSCM', branches: [[name: '*/master']],
			userRemoteConfigs: [[url: 'https://github.com/pravrawa/SampleMvcApp-Pipeline.git]]])
    
    stage 'Build'
            String appsHome     = "C:/jenkins_home/apps"
			
 	      bat "\"${appsHome}/nuget/nuget.exe\" restore SampleWebApp.sln"	     
		  bat "\"C:/Program Files (x86)/MSBuild/14.0/Bin/MSBuild.exe\" SampleWebApp.sln  /p:OutDir=target /p:Configuration=Debug /p:Platform=\"Any CPU\" /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
   
   stage 'Test'      
	     	
	      bat "\"C:/jenkins_home/apps/NUnit.Framework-3.6.0/bin/net-4.5/nunitlite-runner.exe\" --result:TestResult.xml;format=nunit2  Tests/target/Tests.dll"
	      step([$class: 'NUnitPublisher', testResultsPattern:'**/TestResult.xml', debug: false, keepJUnitReports: true, skipJUnitArchiver:false, failIfNoResults: true]) 

		  
   stage 'Code Analysis'
	
	        String sonarMSBuild = "C:/jenkins_home/apps/sonar-scanner-msbuild"			
				
			String sonarqube_host ="http://algsasc2tm0092.r1-core.r1.aig.net:9000/"        
			String projectKey     = "SampleWebApp"
			def currentDir      = pwd()
			
	//		bat "\"${sonarMSBuild}/SonarQube.Scanner.MSBuild.exe\" begin  /d:sonar.host.url=${sonarqube_host}  /d:sonar.login=\"49d8e9a3aabcdb1d51771bd828a8a82e2b50eedf\" /d:sonar.verbose=true /k:\"${projectKey}\" /n:\"${projectKey}\" /v:\"1.0\" "
	//		bat "\"C:/Program Files (x86)/MSBuild/14.0/Bin/MSBuild.exe\" /t:Rebuild"
	//		bat "\"${sonarMSBuild}/MSBuild.SonarQube.Runner.exe\" end"       
     
			bat "\"${sonarMSBuild}/SonarQube.Scanner.MSBuild.exe\" begin /s:${currentDir}/SonarQube.Analysis.xml  /k:\"SampleWebApp\" /n:\"SampleWebApp\" /v:\"1.0\" "
            bat "\"C:/Program Files (x86)/MSBuild/14.0/Bin/MSBuild.exe\" /t:Rebuild"
			bat "\"${sonarMSBuild}/MSBuild.SonarQube.Runner.exe\" end"

   stage 'Archive'
   
			def server = Artifactory.newServer url:'http://algsasc2tm0076.r1-core.r1.aig.net:8081/artifactory', username:'rsingh', password:'AKCp2Vp51q8fNPWgx1Cg2vLGVXBNW4BpKczxnXoePAxCCBMAEaEa9npCcb5TjeYvSM6ttaQqi'
			server.setBypassProxy(true)	    			
				
		    def uploadSpec =
			'''{
			"files": [
				{
					"pattern": "DataLayer/target/**",
					"target": "aig-generic-local/nuget/DataLayer-''' + "${env.BUILD_NUMBER}" + '''.zip"
					
				},
				{
					"pattern": "ServiceLayer/target/**",
					"target": "aig-generic-local/nuget/ServiceLayer-''' + "${env.BUILD_NUMBER}" + '''.zip"
					
				},
				{
					"pattern": "SampleWebApp/target/**",
					"target": "aig-generic-local/nuget/SampleWebApp-''' + "${env.BUILD_NUMBER}" + '''.zip"
					
				}
			]
		}'''

		
		// Upload to Artifactory and publish.		
			def buildInfo = server.upload spec: uploadSpec
			server.publishBuildInfo buildInfo
		
  }		
   catch (err) {

        currentBuild.result = "FAILURE"

            mail body: "It appears that ${env.BUILD_URL} is failing" ,
            from: 'enterprise_jenkins_valicpso@ci.org',
            replyTo: 'girdhar.katiyar@aig.com',
            subject: '${env.JOB_NAME} (${env.BUILD_NUMBER}) failed',
            to: 'Rishi.Singh1@aig.com',
			cc: 'girdhar.katiyar@aig.com,praveen.rawat@aig.com'

        throw err
    }
 
 }
