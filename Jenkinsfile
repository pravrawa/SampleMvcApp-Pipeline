node
{
 try 
 { 
	def workspace = pwd()
    echo $workspace
	stage 'Checkout'
	
			bat 'git init &&  git config http.sslVerify false'
			checkout([$class: 'GitSCM', branches: [[name: '*/master']],
			userRemoteConfigs: [[url: 'https://github.com/pravrawa/SampleMvcApp-Pipeline.git']]])
    
    stage 'Build'
            String appsHome     = "C:/Jenkins/apps"
			
 	      bat "\"${appsHome}/nuget/nuget.exe\" restore SampleWebApp.sln"	     
		  bat "\"C:/Program Files (x86)/MSBuild/14.0/Bin/MSBuild.exe\" SampleWebApp.sln  /p:OutDir=target /p:Configuration=Debug /p:Platform=\"Any CPU\" /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
   
   stage 'Test'      
	     	
	      bat "\"C:/Jenkins/apps/NUnit.Framework-3.6.0/bin/net-4.5/nunitlite-runner.exe\" --result:TestResult.xml;format=nunit2  Tests/target/Tests.dll"
	      step([$class: 'NUnitPublisher', testResultsPattern:'**/TestResult.xml', debug: false, keepJUnitReports: true, skipJUnitArchiver:false, failIfNoResults: true]) 
		  
   stage 'ReportGenerator'
    //  bat "\"C:/Jenkins/apps/ReportGenerator_2.5.2.0/ReportGenerator.exe\" -reports:C:/Jenkins/workspace/sample-project-2/TestResult.xml --targetDir:TestResultsHTML"   
    //  def currentDir2 = pwd()
	//  bat "\"C:/Jenkins/apps/reportunit.1.5.0-alpha1/tools/ReportUnit.exe\" \"${currentDir2}/TestResult.xml\" "
	
	    bat "\"C:/Jenkins/apps/NUnit.HTML.Report.Generator.v1.0/NUnitHTMLReportGenerator.exe\" TestResult.xml TestReportHTML/index.htm "
	
    
	stage 'Publish Results'
		publishHTML (target:[allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'TestReportHTML', reportFiles: 'index.htm', reportName: 'Test Results HTML Report'])
   
   stage 'Code Analysis'
	
	        String sonarMSBuild = "C:/Jenkins/apps/sonar-scanner-msbuild"			
				
			String sonarqube_host ="http://localhost:9000/"        
			String projectKey     = "SampleWebApp"
			def currentDir      = pwd()
			
	//		bat "\"${sonarMSBuild}/SonarQube.Scanner.MSBuild.exe\" begin  /d:sonar.host.url=${sonarqube_host}  /d:sonar.login=\"49d8e9a3aabcdb1d51771bd828a8a82e2b50eedf\" /d:sonar.verbose=true /k:\"${projectKey}\" /n:\"${projectKey}\" /v:\"1.0\" "
	//		bat "\"C:/Program Files (x86)/MSBuild/14.0/Bin/MSBuild.exe\" /t:Rebuild"
	//		bat "\"${sonarMSBuild}/MSBuild.SonarQube.Runner.exe\" end"       
     
			bat "\"${sonarMSBuild}/SonarQube.Scanner.MSBuild.exe\" begin /s:${currentDir}/SonarQube.Analysis.xml  /k:\"SampleWebApp\" /n:\"SampleWebApp\" /v:\"1.0\" "
            bat "\"C:/Program Files (x86)/MSBuild/14.0/Bin/MSBuild.exe\" /t:Rebuild"
			bat "\"${sonarMSBuild}/MSBuild.SonarQube.Runner.exe\" end"
   stage 'Archive'
   
			def server = Artifactory.newServer url:'http://localhost:9090/artifactory', username:'admin', password:'password'
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
		def dirName = 'SampleWebApp-'+"$BUILD_NUMBER.0"
		def zipFileName = 'SampleWebApp-'+"$BUILD_NUMBER.0" + '.zip';
	    //unzip dir:  '''+"${dirName}+''', glob: '', zipFile: '''+"${zipFileName}"+''' 
		//bat "cd download"
		//bat "curl -O http://localhost:9090/artifactory/aig-generic-local/nuget/SampleWebApp-${sampleWebAppVersion}.zip"
		xldCreatePackage artifactsPath: '/', manifestPath: 'deployit-manifest.xml', darPath: '$JOB_NAME-$BUILD_NUMBER.0.dar'
			
  	}  
	 
	stage('Publish') {  
	//	xldPublishPackage serverCredentials: 'Admin', darPath: '$JOB_NAME-$BUILD_NUMBER.0.dar'
        xldPublishPackage darPath: '$JOB_NAME-$BUILD_NUMBER.0.dar', serverCredentials: 'XL-DeployServer'
	}
	
	stage('Deploy') {  
		//xldDeploy serverCredentials: 'Admin', environmentId: 'Environments/Dev', packageId: 'Applications/SampleMvcApp/$BUILD_NUMBER.0'
          xldDeploy environmentId: 'Environments/Dev', packageId: 'Applications/SampleMvcApp/$BUILD_NUMBER.0', serverCredentials: 'XL-DeployServer'
	}  	
			
  }		
   catch (err) {

    //    currentBuild.result = "FAILURE"

    //        mail body: "It appears that ${env.BUILD_URL} is failing" ,
    //        from: 'enterprise_jenkins_valicpso@ci.org',
    //        replyTo: 'girdhar.katiyar@aig.com',
    //        subject: '${env.JOB_NAME} (${env.BUILD_NUMBER}) failed',
    //        to: 'Rishi.Singh1@aig.com',
	//		cc: 'girdhar.katiyar@aig.com,praveen.rawat@aig.com'

        throw err
    }
    
}    
 
