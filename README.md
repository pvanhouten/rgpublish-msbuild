rgpublish-msbuild
=================

MSBuild script for running RGPublish.exe (Red Gate Deployment Manager)

This msbuild script was created to be used with Team Foundation Server (2012).  It mimics the msbuild scripts that are included with the SQL automation pack (CI) for running SqlCI.exe.


Origin - Packaging my web application
-------------------------------------

I had a web application that I wanted to package up and get deployed to my local installation of Red Gate Deployment Manager.  My application is a hybrid MVC4/WebForms application.

The documentation for deployment manager covers packaging applications in a few scenarios (source: [Red Gate](http://documentation.red-gate.com/display/DM2/Packaging+applications)):

1. Publishing from a .nuspec file
2. Publishing from a Visual Studio project file (.csproj)
3. Publishing a directory

I couldn't go with the .nuspec file because I didn't want to be updating it every time I added a file to my project, so I went with option 2.  Fortunately, Red Gate provides a couple tools for the job:

1. Visual Studio Deployment Manager extension
2. RgPublish.exe

The VS extension doesn't really help me in my scenario because I don't want the build coming from my machine, so I opted to use RgPublish.exe.  Unfortunately, I didn't want to be manually running RgPublish.exe from the command line all the time, so I made an msbuild script that I can use with the TFS build services.


Instructions
------------

In source control:
1. Add RgPublish.exe to source control where it can be accessed when the build server pulls down the workspace.  I added it to $/[Team Project]/[Application]/[Branch]/Src/Deploy/RgPublish.exe.

In your web application project:
1. Download the DeploymentManager.targets file and add the file to the root of your web application.
2. Open your web application project file, either:
  - Right click the project, click unload project, then right click the unloaded project and edit the project
  - Open the csproj file using another editor
3. Look for the chunk of import tasks (`<Import Project"..." />`) at the bottom of the file.  They should look something like this:
    <Import Project="$(MSBuildBinPath)\Microsoft.CSharp.targets" />
    <Import Project="$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets" Condition="'$(VSToolsPath)' != ''" />
    <Import Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets" Condition="false" />

  Add this after the last import task:
    <Import Project="DeploymentManager.targets" />
  
  NOTE: Be sure that the DeploymentManager.target file already exists in your project, or it will fail to load when you open your project next time.

In your build (assuming you're using the default template for your builds):
1. Go to the Process section and expand section 3, Advanced.
2. Include these arguments in the MSBuild Arguments section:
    /p:DeployToDM=True /p:RgPublishExecutable=..\..\Deploy\RgPublish.exe /p:CsProjPath=[My CsProj File Name].csproj /p:PackageId=MUAssistant.Web.Edge /p:TargetFeed=http://[Url to your nuget server]/

Here's the breakdown of what all of the arguments mean:
- DeployToDM : This indicates that we want to create a package and publish our application to the deployment manager when a build runs.
- RgPublishExecutable : This is the relative path to the RgPublish executable from the web application.  If you want to use an absolute path, you'll need to tweak the msbuild script.
- CsProjPath : Path to the .csproj file that references the DeploymentManager.targets file, relative to the path of your web application.
- PackageId : The name of the nuget package as it will appear within Deployment Manager
- TargetFeed : The url of the nuget server

Known Issues
------------

1. This script has only been set up to cover packaging from a .csproj file.  If you wanted to package a directory, you'd need to tweak the script.
2. I hate how the pathing is done, but at the moment, it works.  I'd like to change this to use source control paths.  Maybe I'll get to that some day.

