def filepath = "C:/ProgramData/jenkins/.jenkins/workspace/dotnetappscm/aspnet-core-dotnet-core/aspnet-core-dotnet-core.csproj"  //store source file path into a variable filepath
def file = "aspnet-core-dotnet-core/bin/Debug/netcoreapp1.1/publish"   //published file
def targetPath = "D:/jfrog/"              //Stores artifact from jfrog


pipeline 
{
  /*  environment
    {
        appName = "sapna-webapp"
        resourceGroup = "Training-rg"
       
    }*/
    agent any	                  //It tells Jenkins where to execute this Job
	stages                   //It is used to group the tasks
	{
        stage('sonarqube')      //This stage is for code analysis
        {
            steps              //Used to group the step.
            {
                script
                {
                    
                    def scannerHome = tool 'SonarScanner for MSBuild'                           //store the installed tool into a variable
                    withSonarQubeEnv('SonarQubeServer')                                         //it allows you to select the SonarQube server you want to interact
                    {
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" begin /k:\"demo\""
                        
			bat "dotnet build ${filepath}"    
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" end"
                    }
                }
            }
        }
        stage("Quality gate")         //check the status of quality gate 
        {
            steps {
                waitForQualityGate abortPipeline: false
            }
        }
        stage('build')              //Builds the project and its dependencies
        {
            steps
            {
		echo "Builds the project and its dependencies"
                bat "dotnet build ${filepath}"
		   
            }
        }
        stage('test')            //The dotnet test command is used to execute unit tests in a given solution
        {
            steps
            {
                bat "dotnet test ${filepath}"
		    
            }
        }
        stage('publish')       //dotnet publish compiles the application and application is ready for deployment
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
                
			bat "tar.exe -a -c -f WebApp_${BUILD_NUMBER}.zip ${file}"     //it invokes the current build number
                }
        }
        
        stage ('jfrog')      //connects with jenkins
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
        stage('Upload file')      //it manages the File Specifications in a source control
        {
            steps{
                rtUpload (
                 serverId:"Artifactory" ,    //Creates an Artifactory Server Instance named Artifactory
                  spec: '''{
                   "files": [
                      {
                      "pattern": "${WORKSPACE}/WebApp_${BUILD_NUMBER}.zip",
                      "target": "Sapna-dotnet-app"                          // 'Sapna-dotnet-app' is the repository in jfrog             
                      }
                            ]
                           }''',
                        )
            }
        }
        stage ('Publish build info')   //provide url of  for the published build    
        {
            steps 
            {
                rtPublishBuildInfo (
                    serverId: "Artifactory"
                )
            }
        }
stage ('download the artifacts from artifactory')     //This stage downloads artifacts to target path
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
   
	stage('Deploy to azure') //Depoly the published files into the azure webapp
	{
	   steps
		{

			//azureWebAppPublish appName: "${env.appName}", azureCredentialsId: 'Azure', resourceGroup: "${env.resourceGroup}"
			azureWebAppPublish azureCredentialsId: params.Credentials_id , resourceGroup: params.ResourceGroup , appName: params.WebApp
	    }
	}
	}
}

