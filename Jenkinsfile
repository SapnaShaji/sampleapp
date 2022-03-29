def filepath = "C:/ProgramData/jenkins/.jenkins/workspace/dotnetappscm/aspnet-core-dotnet-core/aspnet-core-dotnet-core.csproj"
def file = "aspnet-core-dotnet-core/bin/Debug/netcoreapp1.1/publish"
def targetPath = "D:/jfrog/"


pipeline 
{
    environment
    {
        appName = "sapna-webapp"
        resourceGroup = "Training-rg"
       
    }
    agent any	
	stages 
	{
        stage('sonarqube') 
        {
            steps
            {
                script
                {
                    
                    def scannerHome = tool 'SonarScanner for MSBuild'
                    withSonarQubeEnv('SonarQubeServer') 
                    {
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" begin /k:\"demo\""
                        
			bat "dotnet build ${filepath}"    
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" end"
                    }
                }
            }
        }
        stage("Quality gate") 
        {
            steps {
                waitForQualityGate abortPipeline: false
            }
        }
        stage('build')
        {
            steps
            {
		echo "Builds the project and its dependencies"
                bat "dotnet build ${filepath}"
		   
            }
        }
        stage('test')
        {
            steps
            {
                bat "dotnet test ${filepath}"
		    
            }
        }
        stage('publish')
        {
            steps
            {
                bat "dotnet publish ${filepath}"
		   
            }
        }
        
        stage('Package') 
        {
            steps 
                {
                echo "Deploying to stage environment for more tests!";
                bat "del *.zip"
                
			bat "tar.exe -a -c -f WebApp_${BUILD_NUMBER}.zip ${file}"
                }
        }
        
        stage ('jfrog')
        {
            steps
            {
               rtServer (
                 id: "Artifactory",
                 
                  bypassProxy: true,
                   timeout: 300
                        )
            }
        }
        stage('Upload file')
        {
            steps{
                rtUpload (
                 serverId:"Artifactory" ,
                  spec: '''{
                   "files": [
                      {
                      "pattern": "${WORKSPACE}/WebApp_${BUILD_NUMBER}.zip",
                      "target": "Sapna-dotnet-app"
                      }
                            ]
                           }''',
                        )
            }
        }
        stage ('Publish build info') 
        {
            steps 
            {
                rtPublishBuildInfo (
                    serverId: "Artifactory"
                )
            }
        }
stage ('download the artifacts from artifactory')
        {
            steps
            {
                  rtDownload (
                    serverId: "Artifactory",
                        spec:
                              """{
                                "files": [
                                  {
                                    "pattern": "Sapna-dotnet-app/WebApp_${BUILD_NUMBER}.zip",
                                    "target": "${targetPath}"          
                                  }
                               ]
                              }"""
      )
            }
        }
   /* stage('Extract ZIP') 
	{
	   steps
	   {
		   powershell '''
		                  Expand-Archive 'D:/jfrog/dotnetapp.zip' -DestinationPath 'D:/home/'
		              '''
        } 
	}*/
	stage('Deploy to azure') 
	{
	   steps
		{

			azureWebAppPublish appName: "${env.appName}", azureCredentialsId: 'Azure', resourceGroup: "${env.resourceGroup}"
	    }
	}
	}
}

