AWS CI/CD Web Application Deployment Guide
Day 1: Cloud Web App Setup & Maven
Connecting to Your EC2 Instance
To launch and connect to your EC2 instance from the terminal, you must first secure your private key and then initiate an SSH connection.

Bash
# Secure the PEM file
chmod 400 devops.pem

# Connect to the instance
ssh -i /Users/piyushsharma/Developer/Tutorials/DEVOPS/CI-CD-WEBAPP/devops_piyush.pem ec2-user@ec2-65-1-112-196.ap-south-1.compute.amazonaws.com
Apache Maven Overview
Maven is a build automation and dependency management tool for Java projects. It eliminates the need to manually download .jar files by automatically downloading libraries, compiling code, running tests, and packaging the application (into a JAR or WAR file).

Initializing a Maven Project
The mvn archetype:generate command creates a new project from a template.

Bash
mvn archetype:generate -DgroupId=com.nextwork.app -DartifactId=nextwork-web-project -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false
Command Flags Explained:

-DartifactId: Names your project (e.g., nextwork-web-project).

-DarchetypeArtifactId: Specifies the template type (e.g., maven-archetype-webapp for a web application).

-DinteractiveMode: Set to false to run the installation automatically without pausing for user confirmation.

Day 2: Connect a GitHub Repo with AWS
Since GitHub no longer supports password authentication for Git operations, you must use an SSH key to securely connect your EC2 instance to your repository.

Step 1: Connect to EC2

Bash
ssh -i my-key.pem ec2-user@<EC2_PUBLIC_IP>
Step 2: Generate an SSH Key

Bash
ssh-keygen -t ed25519 -C "ec2-github"
Press Enter to accept the default location (/home/ec2-user/.ssh/id_ed25519). Leave the passphrase empty for automated use.

Step 3: View and Copy the Public Key

Bash
cat ~/.ssh/id_ed25519.pub
Step 4: Add the Key to GitHub

Account-level (Broad access): Go to GitHub Settings → SSH and GPG keys → New SSH key → Paste.

Deploy Key (Secure, repository-level): Go to Repository Settings → Deploy keys → Add deploy key → Paste (Enable "Allow write access" if pushing is required).

Step 5: Test the Connection

Bash
ssh -T git@github.com
Step 6: Clone and Configure

Bash
git clone git@github.com:YOUR_USERNAME/YOUR_REPO.git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
Day 3: Secure Packages with AWS CodeArtifact
AWS CodeArtifact is a secure, central repository for storing external software packages and dependencies.

Core Benefits:

Security: Prevents downloading unverified packages directly from the public internet.

Reliability: Acts as a backup if public package websites experience downtime.

Control: Ensures your entire team uses the exact same version of every package.

IAM Roles vs. IAM Policies

Policy: The rulebook. A document specifying exact allowed or denied actions (e.g., "allow reading from an S3 bucket").

Role: The job position. A container that holds policies. Roles can be assumed temporarily by users, EC2 instances, or other AWS services.

Security Note: When creating an IAM policy for CodeArtifact, the condition "sts:AWSServiceName": "codeartifact.amazonaws.com" ensures the sts:GetServiceBearerToken action is exclusively used for CodeArtifact, preventing unauthorized token generation.

Day 4: Compiling with AWS CodeBuild
AWS CodeBuild is a fully managed continuous integration (CI) tool that compiles source code, runs tests, and produces ready-to-deploy software packages. It eliminates the need to manage idle build servers by charging only for the compute time used.

Project Configuration Options:

Project Type - Default: Manages the entire build process natively within AWS.

Project Type - Runner: Leverages CodeBuild's environment while being orchestrated by external tools like GitHub Actions.

Provisioning - On-demand: Cost-effective. AWS provisions resources only when a build starts and tears them down immediately after.

Provisioning - Reserved: Higher cost, but provides dedicated resources with zero wait times for high-volume teams.

Environment Image: Use a Managed image (AWS-provided template) to skip manual software installation, or a Custom image (your own Docker container) for specialized setups.

Compute: Use EC2 over Lambda for this setup, as Lambda prioritizes speed but does not natively support Java Corretto 8.

The buildspec.yml File
This file must be placed in the root directory of your repository. It acts as the instruction manual for CodeBuild, detailing exactly what commands to run and what tools to install at each stage of the build. Without it, the build will fail.

Day 5: Deployment with CodeDeploy & CloudFormation
AWS CloudFormation (Infrastructure as Code)
CloudFormation uses a single template file to predictably deploy and manage your infrastructure (servers, networks, databases) rather than clicking through the AWS console.

CloudFormation Stack Resources Generated:

VPC (Virtual Private Cloud): The isolated cloud network.

Subnet: A network subdivision for placing resources.

Route Tables: Rules dictating network traffic flow.

Internet Gateway: Bridges your VPC to the public internet.

Security Group: The virtual firewall controlling inbound/outbound traffic.

EC2 Instance: The actual web server hosting the application.

Stack Failure Options:

Roll back all stack resources: Acts as an "undo" button, reverting the environment to its previous state upon failure.

Delete all newly created resources: Prevents orphaned resources from generating unexpected AWS billing charges.

AWS CodeDeploy
A continuous deployment service that automates software rollouts to your servers. It uses an S3 bucket as its "revision location" to fetch the compiled WAR file produced by CodeBuild.

The appspec.yml File
This file lives in your project and tells CodeDeploy exactly how to handle the new files.

version: Formatted as 0.0.

os: Defined as linux.

files: Defines the source (the WAR file) and the destination on the EC2 instance.

hooks: Scripts that run at specific deployment lifecycle stages (BeforeInstall to stop the server, AfterInstall to configure files, ApplicationStart to boot up).

CodeDeploy Architecture & Settings:

Application: The main namespace grouping all configurations for a specific piece of software.

Deployment Group: A specific subset of environments/settings for that application.

IAM Permissions: The AWSCodeDeployRole allows CodeDeploy to interact with EC2, S3 (for build artifacts), Auto Scaling, and CloudWatch (for logging).

Deployment Type: In-place replaces the app on existing instances (causes brief downtime). Blue/green spins up a duplicate environment and routes traffic over once verified (zero downtime).

Targeting: CodeDeploy uses EC2 Tags (e.g., role: webserver) to locate the exact instances that should receive the update.

CodeDeploy Agent: Software running on the EC2 instance that receives and executes instructions from CodeDeploy. (Should be configured to auto-update every 14 days).

Deployment Speed: CodeDeployDefault.AllAtOnce updates every instance simultaneously. It is the fastest route but carries the highest risk if the deployment is faulty.

CodeDeploy's compute platforms are basically the different types of environments where your application can live:



Day 5

EC2/On-premises: This is for traditional server-based applications - like what we're doing in this project. Your app runs on actual servers (either in AWS or in your own data center).

AWS Lambda: This is for serverless applications where you don't manage any servers. Your code just runs when triggered, and AWS handles all the infrastructure.

Amazon ECS: This is for containerized applications running in Docker containers managed by Amazon's Elastic Container Service.

The revision location is the place where CodeDeploy looks to find your application's build artifacts. We're using the S3 bucket that's storing our WAR file, so CodeDeploy knows where to find the latest version of our web app it's deploying to the deployment EC2 instances!

CodePipeline solves these problems by orchestrating your entire workflow, and giving you visibility into the entire process in one place. It can:

✅ Automatically detect code changes in your GitHub repository.
✅ Automatically trigger CodeBuild to build a new project.
✅ Automatically start CodeDeploy with the new build artifacts.
✅ Automatically roll back a change if something fails.

Amazon EC2 (Elastic Compute Cloud) gives you virtual servers in the cloud that you can spin up whenever you need them. Think of it like renting computers in AWS's data centers that you can configure and use without having to worry about the physical hardware.

For this project, we're using EC2 as both the place where we'll deploy our application and the environment where we'll build it initially. It's perfect for this because you can easily SSH into it to make changes and see your application running in real-time.

The Source stage is the very first step in any CI/CD pipeline. Its job is simple but crucial: it fetches the latest version of your code from your chosen repository whenever there are updates. Without this stage, your pipeline would have nothing to build or deploy.

CodePipeline supports various source providers, but for this project, we're using GitHub because that's where our web app's code is stored.

💡 What are Webhook events?
Webhook events let CodePipeline automatically start your pipeline whenever code is pushed to your specified branch in GitHub. This is what makes our pipeline truly "continuous" – it reacts to code changes in real-time!


💡 How do Webhooks work?
Webhooks are like digital notifications. When you enable webhook events, CodePipeline sets up a webhook in your GitHub repository. This webhook is configured to listen for specific events, such as code pushes to the master branch.

Whenever you push code to the master branch, GitHub sends a webhook event (a notification) to CodePipeline. CodePipeline then automatically starts a new pipeline execution in response to this event. It's a seamless way to automate your CI/CD process!