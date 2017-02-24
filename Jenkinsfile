node
{
 try 
 { 
   // adds job parameters within jenkinsfile
   properties([
     parameters([
       stringParam(
         defaultValue: 'C:/Jenkins/apps',
         description: 'Jenkins apps home',
         name: 'appsHome'
       ),
	   stringParam(
         defaultValue: 'https://github.com/pravrawa/SampleMvcApp-Pipeline.git',
         description: 'Project GitHub URL',
         name: 'githubURL'
       ),
	   stringParam(
         defaultValue: 'C:/Program Files (x86)/MSBuild/14.0/Bin',
         description: 'MSBuild Path',
         name: 'MSBuild'
       ),
	   stringParam(
         defaultValue: 'C:/Jenkins/apps/NUnit.Framework-3.6.0/bin/net-4.5',
         description: 'NUnitlite Path',
         name: 'NUnitlite'
       ),
	   stringParam(
         defaultValue: 'C:/Jenkins/apps/sonar-scanner-msbuild',
         description: 'Sonar MSBuild Path',
         name: 'sonarMSBuild'
       ),
	   stringParam(
         defaultValue: 'http://localhost:9000/',
         description: 'Sonar Qube Server Address',
         name: 'sonarqubeHost'
       ),
	   stringParam(
         defaultValue: 'SampleWebApp',
         description: 'Project Key',
         name: 'projectKey'
       ),
	   stringParam(
         defaultValue: 'http://localhost:9090/artifactory',
         description: 'Artifactory Server URL',
         name: 'artifactoryURL'
       ),
	   stringParam(
         defaultValue: 'admin',
         description: 'Artifactory Server user name',
         name: 'artifactoryusername'
       ),
	   stringParam(
         defaultValue: 'password',
         description: 'Artifactory server password',
         name: 'artifactoryPassword'
       ),
       
     ])
   ]) 
  	
	stage 'Checkout'
	
			bat 'git init &&  git config http.sslVerify false'
			checkout([$class: 'GitSCM', branches: [[name: '*/master']],
			userRemoteConfigs: [[url: "${githubURL}"]]])
    
    stage 'Build'
            		
 	      bat "\"${appsHome}/nuget/nuget.exe\" restore SampleWebApp.sln"	     
		  bat "\"${MSBuild}/MSBuild.exe\" SampleWebApp.sln  /p:OutDir=target /p:Configuration=Debug /p:Platform=\"Any CPU\" /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
	
	stage 'Test'      
	     	
	      bat "\"${NUnitlite}/nunitlite-runner.exe\" --result:TestResult.xml;format=nunit2  Tests/target/Tests.dll"
	      step([$class: 'NUnitPublisher', testResultsPattern:'**/TestResult.xml', debug: false, keepJUnitReports: true, skipJUnitArchiver:false, failIfNoResults: true]) 
	
	stage 'Code Analysis'
		    
			def currentDir      = pwd()
			bat "\"${sonarMSBuild}/SonarQube.Scanner.MSBuild.exe\" begin /s:${currentDir}/SonarQube.Analysis.xml  /k:\"SampleWebApp\" /n:\"SampleWebApp\" /v:\"1.0\" "
            bat "\"${MSBuild}/MSBuild.exe\" /t:Rebuild"
			bat "\"${sonarMSBuild}/MSBuild.SonarQube.Runner.exe\" end"	

			
	stage 'Archive'
   
			def server = Artifactory.newServer url:"${artifactoryURL}", username:"${artifactoryusername}", password:"${artifactoryPassword}"
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
	
	stage('Package') {  
		
	     xldCreatePackage artifactsPath: '/', manifestPath: 'deployit-manifest.xml', darPath: '$JOB_NAME-$BUILD_NUMBER.0.dar'
			
  	}  
	
	stage('Publish') {  
	
        xldPublishPackage darPath: '$JOB_NAME-$BUILD_NUMBER.0.dar', serverCredentials: 'XL-DeployServer'
	}
	
	stage('Deploy') {  
	
          xldDeploy environmentId: 'Environments/Dev', packageId: 'Applications/SampleMvcApp/$BUILD_NUMBER.0', serverCredentials: 'XL-DeployServer'
	}  	
	
 }
	catch (err) {

    //    currentBuild.result = "FAILURE"

    //        mail body: "It appears that ${env.BUILD_URL} is failing" ,
    //        from: 'enterprise_jenkins_valicpso@ci.org',
    //        replyTo: 'girdhar.katiyar@abc.com',
    //        subject: '${env.JOB_NAME} (${env.BUILD_NUMBER}) failed',
    //        to: 'Rishi.Singh1@abc.com',
	//		cc: 'girdhar.katiyar@abc.com,praveen.rawat@abc.com'

        throw err
    }
    
}        
     
 
