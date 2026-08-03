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