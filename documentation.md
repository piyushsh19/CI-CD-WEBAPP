DAY 1 : Setup the Web app in the cloud 
chmod 400 devops.pem
to read the pem

ssh -i pathofpem publicnetwork
This way we launch the ec2 instance from the terminal
ssh -i /Users/piyushsharma/Developer/Tutorials/DEVOPS/CI-CD-WEBAPP/devops_piyush.pem ec2-user@ec2-65-1-112-196.ap-south-1.compute.amazonaws.com

Apache Maven is a build automation and dependency management tool for Java projects.

It helps you:

Download libraries (dependencies)
Compile your Java code
Run tests
Package your application into a JAR or WAR
Manage project versions
Standardize project structure

Instead of manually downloading dozens of .jar files and configuring them, Maven does it automatically.

The mvn archetype:generate command specifically tells Maven to create a new project from a template (which Maven calls an archetype). This command sets up a basic structure for your project, so you don't have to start from scratch.


💡 Extra for Experts: Some of the details you've specified in this command are...





-DartifactId=nextwork-web-project names your project



-DarchetypeArtifactId=maven-archetype-webapp specifies that you're creating a web application.



-DinteractiveMode=false runs the command without pausing for user input, so Maven will go ahead and install everything without waiting for your confirmation.

mvn archetype:generate -DgroupId=com.nextwork.app -DartifactId=nextwork-web-project -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false


DAY2 - Connect a GitHub Repo with AWS
on git hub the padsowrd support is not longer suportered

ned to setup using the ssh-keygen -t ed25519 -C "ec2-github"

Option 1 (Recommended): Use an SSH key on the EC2 instance
Step 1: Connect to your EC2 instance
ssh -i my-key.pem ec2-user@<EC2_PUBLIC_IP>

(For Ubuntu AMIs, the user is usually ubuntu instead of ec2-user.)

Step 2: Generate an SSH key
ssh-keygen -t ed25519 -C "ec2-github"

Press Enter for the default location:

/home/ec2-user/.ssh/id_ed25519

You can leave the passphrase empty for automated use.

Step 3: View the public key
cat ~/.ssh/id_ed25519.pub

Copy the entire output.

Step 4: Add the key to GitHub

You have two choices:

Option A: Add it to your GitHub account

GitHub → Settings → SSH and GPG keys
Click New SSH key
Paste the public key

This gives the EC2 instance access to all repositories your account can access.

Option B (more secure): Add it as a Deploy Key

For a single repository:

Repository → Settings → Deploy keys
Add deploy key
Paste the key
Enable Allow write access only if the EC2 instance needs to push.
Step 5: Test the connection
ssh -T git@github.com

The first time, type:

yes

Then you should see:

Hi <your-github-username>! You've successfully authenticated...
Step 6: Clone the repository
git clone git@github.com:YOUR_USERNAME/YOUR_REPO.git

Example:

git clone git@github.com:PiyushSharma/MyAIProject.git

git config --global user.name "enter your name"
git config --global user.email enter your email address

git diff --staged

Day 3 - Secure Packages with CodeArtifact
CodeArtifact is a secure, central place to store all your software packages. When you're building an application, you typically use dozens of external packages or libraries - things other developers have created that you don't want to build from scratch.

An artifact repository gives you a consistent, reliable place to store and retrieve these components. This gives you three big benefits:





1️⃣ Security: Everyone in a team retrieves packages from a secure repository (CodeArtifact), instead of downloading from unsafe sources on the internet (hello, security risks)!



2️⃣ Reliability: If public package websites go down, you have backups in your CodeArtifact repository.



3️⃣ Control: Your team can easily share and use the same versions of packages, instead of everyone working with a different version of the same package.


💡 What is an IAM role?
An IAM role is like a set of permissions that you can assign to your AWS resources.

For our project, we're creating an IAM role specifically for an EC2 instance to get CodeArtifact access, but roles can grant permissions to any AWS service.


💡 What's the difference between a policy and a role?
Think of a policy as the actual list of permissions - it's a document that specifies exactly what actions are allowed or denied on which AWS resources. For example, "allow reading from this S3 bucket" or "allow publishing to CodeArtifact."

A role is the container that holds these policies and can be assumed by users, applications, or AWS services. You attach policies to roles, then assign the role to whoever needs those permissions.

This separation is powerful because:





You can attach the same policy to multiple roles



A role can have multiple policies attached



You can modify a policy once and affect all roles using it



Roles can be assumed temporarily, while policies define the permanent permission boundaries

It's like the difference between writing down rules (policies) and creating a job position (role) that follows those rules. The position can be filled by different people or services, but the rules remain consistent.

"sts:AWSServiceName": "codeartifact.amazonaws.com": This condition ensures that the sts:GetServiceBearerToken action is only allowed when the AWS service name is codeartifact.amazonaws.com. This is a security measure to restrict the use of this STS action specifically for CodeArtifact.

[Important] an IAM policy that will allow EC2 instances to access CodeArtifact. In the next step, we'll attach this policy to an IAM role and then associate that role with your EC2 instance. 

DAY4 -
💡 What is AWS CodeBuild?
AWS CodeBuild is a fully build tool for your code. It takes your source code, compiles it, runs tests, and packages it up. Engineers love continuous integration tools like CodeBuild because you don't have to manually set up and manage any build servers yourself, and you only pay for the compute time you use for building your projects (instead of entire servers that are idle most of the time). Think of it as a super-efficient, scalable, and managed service that handles all the heavy lifting of building and testing your applications.

Continuous Integration is like having a quality control checkpoint that automatically kicks in whenever anyone on your team makes changes to your code. Instead of waiting until the end of a project to discover that something broke, CI helps you catch and fix issues early and often. CI helps you constantly check that everything still works as expected - running tests, compiling code, and making sure new changes play nicely with the existing codebase.

Default project: This is your standard option that most teams use. It's perfect when you want to manage your entire build process within AWS. You get full control over how your build runs, what goes in, and what comes out - all without leaving the AWS ecosystem.



Runner project: This option is for teams who already have CI systems like GitHub Actions or GitLab CI but want to tap into the power of CodeBuild's build environment. It's like having CodeBuild do the heavy lifting while your existing CI system orchestrates the overall process.

💡 Why did we pick on-demand?
Provisioning model determines how AWS will set up and manage everything needed for your build. Choosing On-demand means AWS will create the resources you need for your build only when you start it, and tear them down when the build is done. This is cost-effective and efficient!





Reserved capacity gives you dedicated build resources always at your disposal. It costs more overall but gives you consistent performance and no wait times. Great for teams that are building constantly throughout the day. If on-demand is like ordering a taxi, reserved capacity is like renting your own car that you can access anytime you need it.

Environment image is like a template for your build environment (just like how AMI's are templates for your EC2 instances). In more technical terms, environment images are pre-configured versions of the build environment so you won't need to install all the software/tools/settings required to build a project. We choose Managed image here, which means we're using a template that AWS has already created for us. The next few settings we pick underneath this will then tell CodeBuild what kind of image we're looking for.

Custom image lets you bring your own Docker image with exactly the tools and configurations your project needs. It's like designing your own custom workspace from scratch - more work to set up, but perfectly tailored to your specific requirements.
Compute sets up the servers that will actually run the commands and do the work for your project's build! Your project's build will run on Amazon EC2 instances, which are more flexible and powerful than AWS Lambda functions.

Note: Lambda is optimized for speed and faster startup times. Our web app is fairly simple and doesn't require a lot of resources, but it's also built in Java Corretto 8, which is a language that's not supported by Lambda.

The buildspec.yml file is like a detailed instruction manual for CodeBuild. Placed in the root of your repository, it tells CodeBuild exactly what to do at each stage of the build process - what tools to install, what commands to run, and what files to package up when it's done.

CodeBuild automatically looks for a file named buildspec.yml in the root directory of your source code. If it finds one, it uses it to execute the build. If not, the build will fail (as we'll see later) 👀

Day 5

✅ VS Code and GitHub and to write and store your web app's code.



✅ CodeArtifact to secure web app's packages.



✅ CodeBuild to compile and package your app into a handy WAR file for deployment.



⏭️ Now, AWS CodeDeploy is here to deploy that file on web servers, so users can see your web app!



Get ready to:





☁️ Launch a deployment environment using AWS CloudFormation.



⚙️ Write deployment scripts to automate deployment commands.



🚀 Deploy your web app with CodeDeploy and see it live!



💎 Implement a disaster recovery technique - roll back a deployment!



💡 Why am I learning about AWS CodeDeploy?
When you're developing software, you need a reliable way to release new versions of your app to the world.

AWS CodeDeploy is a continuous deployment service, which means it automates how you get new software versions onto your servers. Instead of manually moving files and restarting services yourself, CodeDeploy automatically runs a deployment using settings and commands that you define.

As part of a CI/CD pipeline, CodeDeploy makes software releases faster, more consistent, and way less stressful.


💡 What is AWS CloudFormation?
Think of CloudFormation is AWS' infrastructure as code tool. Instead of clicking around the AWS console to set up resources (which gets tedious fast!), you write a single template file that describes everything you need - your EC2 instances, security groups, databases, and more. Then, CloudFormation reads this file and builds your entire environment for you, exactly the same way every time.


💡 What is Infrastructure as Code?
Just like software developers write code to build applications, Infrastructure as Code (IaC) is a type of software that lets you write code to create your servers, networks, and other infrastructure. Instead of manually configuring each server (time-consuming and error-prone), you have a script that sets everything up perfectly every time. Your infrastructure becomes predictable, repeatable, and much easier to manage!

What are Stack failure options?
Stack failure options are your safety net when things don't go as planned. They determine what CloudFormation should do if it runs into an error while creating your resources:





Roll back all stack resources: This is like having an "undo" button for your entire deployment. If anything fails, CloudFormation will automatically revert everything back to how it was before you started. This prevents you from ending up with a half-built environment that might not work or cost you money unnecessarily.



Delete all newly created resources: This makes sure CloudFormation cleans up after itself during a rollback. No resources left behind to surprise you on your bill next month!

IAM roles are like special visitor passes that AWS services can "wear" to temporarily access other services. In our case, our deployment EC2 instance will use a role to access files from S3.

Hmmm... why do you think it'll need access to S3 (have you created an S3 bucket anywhere)? Aha - your deployment environment will need to use the build artifacts stored in your S3 bucket!

💡 What resources are being created by the CloudFormation template?
The CloudFormation template is creating a:





VPC (Virtual Private Cloud): A virtual network in the cloud for your AWS resources.



Subnet: A subdivision of the VPC where you can place resources.



Route Tables: Define how network traffic is routed within the VPC.



Internet Gateway: Allows resources in your VPC to connect to the internet.



Security Group: Acts as a virtual firewall to control inbound and outbound traffic for your EC2 instance.



EC2 Instance: The virtual server where your web application will be deployed.


💡 Why are we deploying networking resources too?
By defining these networking resources in the template, we're not just launching an EC2 instance, but creating a complete, secure, and configurable infrastructure that can be easily replicated or modified. This is an especially good idea for EC2 instances that are hosting web apps, because they have more complex needs like connecting with multiple databases and controlling both public and private network traffic.

💡 What does each section in appspec.yml mean?
The appspec.yml file is essentially the instruction manual for CodeDeploy. Here's what each part does:





version: 0.0: This is just the format version CodeDeploy expects (yes, starting at 0.0 is a bit odd).



os: linux: Tells CodeDeploy we're deploying to a Linux system, not Windows.



files: This section maps what files go where:





source: /target/nextwork-web-project.war: "Take this WAR file from our artifact..."



destination: /usr/share/tomcat10/webapps/: "...and put it here on the EC2 instance."



hooks: These are like event triggers that run at specific points in the deployment:





BeforeInstall: "Before you install the new version, run this script" - we're stopping the server first.



AfterInstall: "After the files are copied, run this" - we're installing dependencies.



ApplicationStart: "Once everything's ready, run this" - we're starting the server.



Each hook specifies the script location, a timeout (5 minutes max), and that it should run as the root user.


Think of appspec.yml as the choreographer that ensures everyone moves in the right sequence during deployment.

