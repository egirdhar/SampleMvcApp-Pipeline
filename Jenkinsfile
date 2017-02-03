node
{
	stage 'Checkout'
	
			bat 'git init &&  git config http.sslVerify false'
			checkout([$class: 'GitSCM', branches: [[name: '*/master']],
			userRemoteConfigs: [[url: 'https://github.com/egirdhar/SampleMvcApp-Pipeline.git']]])
	
	stage 'Build'
			// specify the home path of exe files
			String appsHome     = "C:/Program Files (x86)/Jenkins/apps"
			String msBuildHome  = "C:/Program Files (x86)/MSBuild/14.0/Bin"
		
			// nuget to download dependencies 
			 bat "\"${appsHome}/nuget.exe\" restore SampleWebApp.sln"			
			 bat "\"${msBuildHome}/MSBuild.exe\" SampleWebApp.sln  /p:OutDir=target /p:Configuration=Debug /p:Platform=\"Any CPU\" /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
	
	
	stage 'Test'
			
			String nUnit        = "${appsHome}/NUnit.org"
			
			bat "\"${nUnit}/nunit-console/nunit3-console.exe\" --result:TestResult.xml;format=nunit2  Tests/target/Tests.dll"
		//	step([$class: 'NUnitPublisher', testResultsPattern:'**/TestResult.xml', debug: false, keepJUnitReports: true, skipJUnitArchiver:false, failIfNoResults: true]) 
	
	stage 'Code Analysis'
	
			String sonarMSBuild = "${appsHome}/sonar-scanner-msbuild-2.2.0.24"			
				
			String sonarqube_host ="http://localhost:9000/"        
			String projectKey     = "SampleWebApp"
			
			bat "\"${sonarMSBuild}/SonarQube.Scanner.MSBuild.exe\" begin  /d:sonar.host.url=${sonarqube_host}  /k:\"${projectKey}\" /n:\"${projectKey}\" /v:\"1.0\" "
			bat "\"${msBuildHome}/MSBuild.exe\" /t:Rebuild"
			bat "\"${sonarMSBuild}/MSBuild.SonarQube.Runner.exe\" end"
				 
			def response = httpRequest "${sonarqube_host}/api/qualitygates/project_status?projectKey=${projectKey}"
		//	def slurper  = new groovy.json.JsonSlurper()
		//	def result   = slurper.parseText(response.content)
		//	println ('Status: '+response.status)
		//	println ('Response: '+response.content)
				
	
	  stage 'Archive'
	
			archiveArtifacts artifacts: 'DataLayer/target/*.*,ServiceLayer/target/*.*,SampleWebApp/target/*.*', fingerprint: true
			def server = Artifactory.newServer url:'http://localhost:9090/artifactory', username:'admin', password:'password'
			server.setBypassProxy(true)
		
			def uploadSpec =
			'''{
			"files": [
				{
					"pattern": "DataLayer/target/**",
					"target": "libs-snapshot-local/DataLayer-$BUILD_NUMBER.zip"
					
				},
				{
					"pattern": "ServiceLayer/target/**",
					"target": "libs-snapshot-local/ServiceLayer-.$BUILD_NUMBER.zip"
					
				},
				{
					"pattern": "SampleWebApp/target/**",
					"target": "libs-snapshot-local/SampleWebApp-.$BUILD_NUMBER.zip"
					
				}
			]
		}'''
		
		// Upload to Artifactory and publish.		
			def buildInfo1 = server.upload spec: uploadSpec
			server.publishBuildInfo buildInfo1 
			
}

  