# Deploying an ASP\.NET Core Application with AWS Elastic Beanstalk<a name="dotnet-core-tutorial"></a>

In this tutorial, you walk through the process of building a new ASP\.NET Core application and deploying it to Elastic Beanstalk\. You use the \.NET Core SDK's `dotnet` command line tool to generate a basic command line \.NET Core application, install dependencies, compile code, and run applications locally\.

Next, you modify the default `Program` class, and add an ASP\.NET `Startup` class and configuration files to make an application that serves HTTP requests with ASP\.NET and IIS\. The `dotnet publish` command generates compiled classes and dependencies that you can bundle with a `web.config` file to create a *site archive* that you can deploy to an Elastic Beanstalk environment\.

Elastic Beanstalk uses a deployment manifest to configure deployments for \.NET Core applications, custom applications, and multiple \.NET Core or MSBuild applications on a single server\. To deploy a \.NET Core application to a Windows Server environment, you add the site archive to an application source bundle with a deployment manifest\. The deployment manifest tells Elastic Beanstalk the path at which the site should run and can be used to configure application pools and run multiple applications at different paths\.

**Note**  
The application source code is available here: [dotnet\-core\-tutorial\-source\.zip](samples/dotnet-core-tutorial-source.zip)  
The deployable source bundle is available here: [dotnet\-core\-tutorial\-bundle\.zip](samples/dotnet-core-tutorial-bundle.zip)


+ [Prerequisites](#dotnet-core-tutorial-prereqs)
+ [Generate a \.NET Core Project](#dotnet-core-tutorial-generate)
+ [Launch an Elastic Beanstalk Environment](#dotnet-core-tutorial-launch)
+ [Update the Source Code](#dotnet-core-tutorial-update)
+ [Deploy Your Application](#dotnet-core-tutorial-deploy)
+ [Clean Up](#w3ab1c39c31c44)
+ [Next Steps](#dotnet-core-tutorial-nextsteps)

## Prerequisites<a name="dotnet-core-tutorial-prereqs"></a>

This tutorial uses the \.NET Core SDK to generate a basic \.NET Core application, run it locally, and build a deployable package\.

**Requirements**

+ \.NET Core \(x64\) 1\.0\.1, 2\.0\.0, or newer 

**To install the \.NET Core SDK**

1. Download the installer from [microsoft\.com/net/core](https://www.microsoft.com/net/core#windows)\. Choose **Windows**, then under **Select your environment** choose **Command line / other**\. Choose **Download \.NET Core SDK**\.

1. Run the installer and follow the instructions\.

This tutorial uses a command line ZIP utility to create a source bundle that you can deploy to Elastic Beanstalk\. To use the `zip` command in Windows, you can install `UnxUtils`, a lightweight collection of useful command line utilities like `zip` and `ls`\. \(Alternatively, you can use Windows Explorer or any other ZIP utility to create source bundle archives\.\)

**To install UnxUtils**

1. Download `[UnxUtils](https://sourceforge.net/projects/unxutils/)`\.

1. Extract the archive to a local directory\. For example, `C:\Program Files (x86)`\.

1. Add the path to the binaries to your Windows PATH user variable\. For example, `C:\Program Files (x86)\UnxUtils\usr\local\wbin`\.

1. Open a new command prompt window and run the `zip` command to verify that it works:

   ```
   > zip
   Copyright (C) 1990-1999 Info-ZIP
   Type 'zip "-L"' for software license.
   ...
   ```

## Generate a \.NET Core Project<a name="dotnet-core-tutorial-generate"></a>

Use the `dotnet` command line tool to generate a new C\# \.NET Core project and run it locally\. The default \.NET Core application is a command line utility that prints `Hello World!` and then exits\. 

**To generate a new \.NET Core project**

1. Open a new command prompt window and navigate to your user folder\.

   ```
   > cd %USERPROFILE%
   ```

1. Use the `dotnet new` command to generate a new \.NET Core project\.

   ```
   C:\Users\username> dotnet new console -o dotnet-core-tutorial
   Content generation time: 65.0152 ms
   The template "Console Application" created successfully.
   C:\Users\username> cd dotnet-core-tutorial
   ```

1. Use the `dotnet restore` command to install dependencies\.

   ```
   C:\Users\username\dotnet-core-tutorial> dotnet restore
   Restoring packages for C:\Users\username\dotnet-core-tutorial\dotnet-core-tutorial.csproj...
   Generating MSBuild file C:\Users\username\dotnet-core-tutorial\obj\dotnet-core-tutorial.csproj.nuget.g.props.
   Generating MSBuild file C:\Users\username\dotnet-core-tutorial\obj\dotnet-core-tutorial.csproj.nuget.g.targets.
   Writing lock file to disk. Path: C:\Users\username\dotnet-core-tutorial\obj\project.assets.json
   Restore completed in 1.25 sec for C:\Users\username\dotnet-core-tutorial\dotnet-core-tutorial.csproj.
   
   NuGet Config files used:
       C:\Users\username\AppData\Roaming\NuGet\NuGet.Config
       C:\Program Files (x86)\NuGet\Config\Microsoft.VisualStudio.Offline.config
   Feeds used:
       https://api.nuget.org/v3/index.json
       C:\Program Files (x86)\Microsoft SDKs\NuGetPackages\
   ```

1. Use the `dotnet run` command to build and run the application locally\.

   ```
   C:\Users\username\dotnet-core-tutorial> dotnet run
   Hello World!
   ```

The default application prints `Hello World!` to the console and exits\. Before you deploy the application to Elastic Beanstalk, you update it to serve HTTP requests with ASP\.NET and IIS\.

## Launch an Elastic Beanstalk Environment<a name="dotnet-core-tutorial-launch"></a>

Use the AWS Management Console to launch an Elastic Beanstalk environment\. Choose the **Windows Server 2012R2 v1\.2\.0** platform configuration and accept the default settings and sample code\. After you launch and configure your environment, you can deploy new source code at any time\.

**To launch an environment \(console\)**

1. Open the Elastic Beanstalk console using this preconfigured link: [console\.aws\.amazon\.com/elasticbeanstalk/home\#/newApplication?applicationName=tutorials&environmentType=LoadBalanced](https://console.aws.amazon.com/elasticbeanstalk/home#/newApplication?applicationName=tutorials&environmentType=LoadBalanced)

1. For **Platform**, choose the platform that matches the language used by your application\.

1. For **Application code**, choose **Sample application**\.

1. Choose **Review and launch**\.

1. Review all options\. When you're satisfied with them, choose **Create app**\.

Environment creation takes about 10 minutes\. During this time you can update your source code\.

## Update the Source Code<a name="dotnet-core-tutorial-update"></a>

Update the default application to use ASP\.NET and IIS\. ASP\.NET is the website framework for \.NET\. IIS is the web server that runs the application on the EC2 instances in your Elastic Beanstalk environment\.

**Note**  
The source code is available here: [dotnet\-core\-tutorial\-source\.zip](samples/dotnet-core-tutorial-source.zip)

**To add ASP\.NET and IIS support to your code**

1. Update `Program.cs` to run a web host builder\.  
**Example c:\\users\\username\\dotnet\-core\-tutorial\\Program\.cs**  

   ```
   using System;
   using Microsoft.AspNetCore.Hosting;
   using System.IO;
   
   namespace aspnetcoreapp
   {
       public class Program
       {
           public static void Main(string[] args)
           {
               var host = new WebHostBuilder()
                 .UseKestrel()
                 .UseContentRoot(Directory.GetCurrentDirectory())
                 .UseIISIntegration()
                 .UseStartup<Startup>()
                 .Build();
   
               host.Run();
           }
       }
   }
   ```

1. Add a `Startup.cs` file to run an ASP\.NET website\.  
**Example c:\\users\\username\\dotnet\-core\-tutorial\\Startup\.cs**  

   ```
   using System;
   using Microsoft.AspNetCore.Builder;
   using Microsoft.AspNetCore.Hosting;
   using Microsoft.AspNetCore.Http;
   
   namespace aspnetcoreapp
   {
       public class Startup
       {
           public void Configure(IApplicationBuilder app)
           {
               app.Run(context =>
               {
                   return context.Response.WriteAsync("Hello from ASP.NET Core!");
               });
           }
       }
   }
   ```

1. Add a `web.config` file to configure the IIS server\.  
**Example c:\\users\\username\\dotnet\-core\-tutorial\\web\.config**  

   ```
   <?xml version="1.0" encoding="utf-8"?>
   <configuration>
     <system.webServer>
       <handlers>
         <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
       </handlers>
       <aspNetCore processPath="dotnet" arguments=".\dotnet-core-tutorial.dll" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout" forwardWindowsAuthToken="false" />
     </system.webServer>
   </configuration>
   ```

1. Update `dotnet-core-tutorial.csproj` to include IIS middleware and include the `web.config` file in the output of `dotnet publish`\.  
**Example c:\\users\\username\\dotnet\-core\-tutorial\\dotnet\-core\-tutorial\.csproj**  

   ```
   <Project Sdk="Microsoft.NET.Sdk">
   
       <PropertyGroup>
           <OutputType>Exe</OutputType>
           <TargetFramework>netcoreapp1.1</TargetFramework>
       </PropertyGroup>
   
       <ItemGroup>
           <PackageReference Include="Microsoft.AspNetCore.Server.Kestrel" />
       </ItemGroup>
   
       <ItemGroup>
           <PackageReference Include="Microsoft.AspNetCore.Server.IISIntegration" />
       </ItemGroup>
   
       <ItemGroup>
           <None Include="web.config" CopyToPublishDirectory="Always" />
       </ItemGroup>
   
   </Project>
   ```

Next, you install the new dependencies and run the ASP\.NET website locally\.

**To run the website locally**

1. Use the `dotnet restore` command to install dependencies\.

1. Use the `dotnet run` command to build and run the app locally\.

1. Open [localhost:5000](http://localhost:5000) to view the site\.

To run the application on a web server, you need to bundle the compiled source code with a `web.config` configuration file and runtime dependencies\. The `dotnet` tool provides a `publish` command that gathers these files in a directory based on the configuration in `dotnet-core-tutorial.csproj`\.

**To build your website**

+ Use the `dotnet publish` command to output compiled code and dependencies to a folder named `site`\.

  ```
  C:\users\username\dotnet-core-tutorial> dotnet publish -o site
  ```

To deploy the application to Elastic Beanstalk, bundle the site archive with a deployment manifest that tells Elastic Beanstalk how to run it\.

**To create a source bundle**

1. Add the files in the site folder to a ZIP archive\.

   ```
   C:\users\username\dotnet-core-tutorial> cd site
   C:\users\username\dotnet-core-tutorial\site> zip ../site.zip *
     adding: dotnet-core-tutorial.deps.json (164 bytes security) (deflated 81%)
     adding: dotnet-core-tutorial.dll (164 bytes security) (deflated 58%)
     adding: dotnet-core-tutorial.pdb (164 bytes security) (deflated 30%)
     adding: dotnet-core-tutorial.runtimeconfig.json (164 bytes security) (deflated 25%)
     adding: Microsoft.AspNetCore.Hosting.Abstractions.dll (164 bytes security) (deflated 48%)
     adding: Microsoft.AspNetCore.Hosting.dll (164 bytes security) (deflated 54%)
     adding: Microsoft.AspNetCore.Hosting.Server.Abstractions.dll (164 bytes security) (deflated 45%)
     adding: Microsoft.AspNetCore.Http.Abstractions.dll (164 bytes security) (deflated 52%)
     adding: Microsoft.AspNetCore.Http.dll (164 bytes security) (deflated 55%)
     adding: Microsoft.AspNetCore.Http.Extensions.dll (164 bytes security) (deflated 50%)
     adding: Microsoft.AspNetCore.Http.Features.dll (164 bytes security) (deflated 50%)
     adding: Microsoft.AspNetCore.HttpOverrides.dll (164 bytes security) (deflated 47%)
     adding: Microsoft.AspNetCore.Server.IISIntegration.dll (164 bytes security) (deflated 47%)
     adding: Microsoft.AspNetCore.Server.Kestrel.dll (164 bytes security) (deflated 62%)
     adding: Microsoft.AspNetCore.WebUtilities.dll (164 bytes security) (deflated 55%)
     adding: Microsoft.Extensions.Configuration.Abstractions.dll (164 bytes security) (deflated 48%)
     adding: Microsoft.Extensions.Configuration.dll (164 bytes security) (deflated 45%)
     adding: Microsoft.Extensions.Configuration.EnvironmentVariables.dll (164 bytes security) (deflated 47%)
     adding: Microsoft.Extensions.DependencyInjection.Abstractions.dll (164 bytes security) (deflated 55%)
     adding: Microsoft.Extensions.DependencyInjection.dll (164 bytes security) (deflated 50%)
     adding: Microsoft.Extensions.FileProviders.Abstractions.dll (164 bytes security) (deflated 46%)
     adding: Microsoft.Extensions.FileProviders.Physical.dll (164 bytes security) (deflated 46%)
     adding: Microsoft.Extensions.FileSystemGlobbing.dll (164 bytes security) (deflated 48%)
     adding: Microsoft.Extensions.Logging.Abstractions.dll (164 bytes security) (deflated 55%)
     adding: Microsoft.Extensions.Logging.dll (164 bytes security) (deflated 43%)
     adding: Microsoft.Extensions.ObjectPool.dll (164 bytes security) (deflated 45%)
     adding: Microsoft.Extensions.Options.dll (164 bytes security) (deflated 46%)
     adding: Microsoft.Extensions.PlatformAbstractions.dll (164 bytes security) (deflated 45%)
     adding: Microsoft.Extensions.Primitives.dll (164 bytes security) (deflated 49%)
     adding: Microsoft.Net.Http.Headers.dll (164 bytes security) (deflated 52%)
     adding: System.Diagnostics.Contracts.dll (164 bytes security) (deflated 46%)
     adding: System.Diagnostics.StackTrace.dll (164 bytes security) (deflated 45%)
     adding: System.Net.WebSockets.dll (164 bytes security) (deflated 47%)
     adding: System.Runtime.CompilerServices.Unsafe.dll (164 bytes security) (deflated 42%)
     adding: System.Text.Encodings.Web.dll (164 bytes security) (deflated 57%)
     adding: web.config (164 bytes security) (deflated 39%)
   C:\users\username\dotnet-core-tutorial\site> cd ../
   ```

1. Add a deployment manifest that points to the site archive\.  
**Example c:\\users\\username\\dotnet\-core\-tutorial\\aws\-windows\-deployment\-manifest\.json**  

   ```
   {
       "manifestVersion": 1,
       "deployments": {
           "aspNetCoreWeb": [
           {
               "name": "test-dotnet-core",
               "parameters": {
                   "appBundle": "site.zip",
                   "iisPath": "/",
                   "iisWebSite": "Default Web Site"
               }
           }
           ]
       }
   }
   ```

1. Use the `zip` command to create a source bundle named `dotnet-core-tutorial.zip`\.

   ```
   C:\users\username\dotnet-core-tutorial> zip dotnet-core-tutorial.zip site.zip aws-windows-deployment-manifest.json
     adding: site.zip (164 bytes security) (stored 0%)
     adding: aws-windows-deployment-manifest.json (164 bytes security) (deflated 50%)
   ```

## Deploy Your Application<a name="dotnet-core-tutorial-deploy"></a>

Deploy the source bundle to the Elastic Beanstalk environment that you created earlier\.

**Note**  
You can download the source bundle here: [dotnet\-core\-tutorial\-bundle\.zip](samples/dotnet-core-tutorial-bundle.zip)

**To deploy a source bundle**

1. Open the [Elastic Beanstalk console](https://console.aws.amazon.com/elasticbeanstalk)\.

1. Navigate to the management page for your environment\.

1. Choose **Upload and Deploy**\.

1. Choose **Choose File** and use the dialog box to select the source bundle\.

1. Choose **Deploy**\.

1. When the deployment completes, choose the site URL to open your website in a new tab\.

The application simply writes `Hello from ASP.NET Core!` to the response and returns\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/dotnet-core-tutorial-site.png)

Launching an environment creates the following resources:

+ **EC2 instance** – An Amazon Elastic Compute Cloud \(Amazon EC2\) virtual machine configured to run web apps on the platform that you choose\.

  Each platform runs a different set of software, configuration files, and scripts to support a specific language version, framework, web container, or combination thereof\. Most platforms use either Apache or nginx as a reverse proxy that sits in front of your web app, forwards requests to it, serves static assets, and generates access and error logs\.

+ **Instance security group** – An Amazon EC2 security group configured to allow ingress on port 80\. This resource lets HTTP traffic from the load balancer reach the EC2 instance running your web app\. By default, traffic is not allowed on other ports\.

+ **Load balancer** – An Elastic Load Balancing load balancer configured to distribute requests to the instances running your application\. A load balancer also eliminates the need to expose your instances directly to the Internet\.

+ **Load balancer security group** – An Amazon EC2 security group configured to allow ingress on port 80\. This resource lets HTTP traffic from the Internet reach the load balancer\. By default, traffic is not allowed on other ports\.

+ **Auto Scaling group** – An Auto Scaling group configured to replace an instance if it is terminated or becomes unavailable\.

+ **Amazon S3 bucket** – A storage location for your source code, logs, and other artifacts that are created when you use Elastic Beanstalk\.

+ **Amazon CloudWatch alarms** – Two CloudWatch alarms that monitor the load on the instances in your environment and are triggered if the load is too high or too low\. When an alarm is triggered, your Auto Scaling group scales up or down in response\.

+ **AWS CloudFormation stack** – Elastic Beanstalk uses AWS CloudFormation to launch the resources in your environment and propagate configuration changes\. The resources are defined in a template that you can view in the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation)\.

+ **Domain name** – A domain name that routes to your web app in the form **subdomain*\.*region*\.elasticbeanstalk\.com*\.

All of these resources are managed by Elastic Beanstalk\. When you terminate your environment, Elastic Beanstalk terminates all the resources that it contains\.

**Note**  
The S3 bucket that Elastic Beanstalk creates is shared between environments and is not deleted during environment termination\. For more information, see \.

## Clean Up<a name="w3ab1c39c31c44"></a>

When you finish working with Elastic Beanstalk, you can terminate your environment\. Elastic Beanstalk terminates all AWS resources associated with your environment, such as Amazon EC2 instances, database instances, load balancers, security groups, and alarms\. 

**To terminate your Elastic Beanstalk environment**

1. Open the [Elastic Beanstalk console](https://console.aws.amazon.com/elasticbeanstalk)\.

1. Navigate to the management page for your environment\.

1. Choose **Actions**, and then choose **Terminate Environment**\.

1. In the **Confirm Termination** dialog box, type the environment name, and then choose **Terminate**\.

In addition, you can terminate database resources that you created outside of your Elastic Beanstalk environment\. When you terminate an Amazon RDS database instance, you can take a snapshot and restore the data to another instance later\.

**To terminate your RDS DB instance**

1. Open the [Amazon RDS console](https://console.aws.amazon.com/rds)\.

1. Choose **Instances**\.

1. Choose your DB instance\.

1. Choose **Instance Actions**, and then choose **Delete**\.

1. Choose whether to create a snapshot, and then choose **Delete**\.

**To delete a DynamoDB table**

1. Open the [Tables page](https://console.aws.amazon.com/dynamodb/home?#tables:) in the DynamoDB console\.

1. Select a table\.

1. Choose **Actions**, and then choose **Delete table**\.

1. Choose **Delete**\.

## Next Steps<a name="dotnet-core-tutorial-nextsteps"></a>

As you continue to develop your application, you'll probably want to manage environments and deploy your application without manually creating a \.zip file and uploading it to the Elastic Beanstalk console\. The Elastic Beanstalk Command Line Interface \(EB CLI\) provides easy\-to\-use commands for creating, configuring, and deploying applications to Elastic Beanstalk environments from the command line\.

If you use Visual Studio to develop your application, you can also use the AWS Toolkit for Visual Studio to deploy changed, manage your Elastic Beanstalk environments, and manage other AWS resources\. See  for more information\.

For developing and testing, you might want to use Elastic Beanstalk's functionality for adding a managed DB instance directly to your environment\. For instructions on setting up a database inside your environment, see \.

Finally, if you plan to use your application in a production environment, configure a custom domain name for your environment and enable HTTPS for secure connections\.