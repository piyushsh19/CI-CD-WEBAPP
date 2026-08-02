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
