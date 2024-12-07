# Lab 1 - Create the Code Repository

As AWS CodeCommit has been deprecated you may not be able to use it in your account if you have not previously used it. There are 2 options for this lab:

***Option 1*** uses CodeCommit for those who have access (though using option 2 is recommended)
***Option 2*** uses GitHub with a connector to AWS CodePipeline

After the initial setup of the code repository the remaining instructions are essentially the same


## Option 1 - Create an AWS CodeCommit Repository

1. Create HTTPS Git credentials for AWS CodeCommit
- Go to your user account in IAM and the Security credentials tab
- Generate HTTPS Git credentials for AWS CodeCommit and store the output

2. Create an AWS CodeCommit repository:
- Go to the AWS Management Console and navigate to the CodeCommit service
- Click on "Create repository" and provide a name for your repository
- Optionally, you can add a description and choose any additional settings
- Click on "Create" to create the repository

3. Clone the repository to your local machine (or use AWS CloudShell):
- Open a terminal or command prompt on your local machine
- Run the following command to clone the repository:

```
git clone <repository-clone-url>
```
- Replace `<repository-clone-url>` with the clone URL of your CodeCommit repository

4. Commit the code changes and push them to AWS CodeCommit
- In the terminal or command prompt, navigate to the cloned repository directory
- Copy the following files/folder into the directory

appspec.yml
index.html
scripts
    |_ install_dependencies
    |_ start_server
    |_ stop_server

- Run the following commands:
   
```bash
git add .
git commit -m "Initial commit"
git push
```
- Enter your CodeCommit credentials if prompted

## Option 2 - Create a GitHub Repository and Connect to AWS CodePipeline

1. In your GitHub account create a new repository
- Click "new' and enter a repository name
- Set the repository to public
- Click "Create repository"

2. Clone the repository
- Copy the HTTPS repository URL from GitHub
- Run the following command to clone the repository:

```
git clone <repository-clone-url>
```

3. Add code / files and commit
- In the terminal or command prompt, navigate to the cloned repository directory
- Copy the following files/folder into the directory

appspec.yml
index.html
scripts
    |_ install_dependencies
    |_ start_server
    |_ stop_server

- Run the following commands:
   
```bash
git add .
git commit -m "Initial commit"
git push
```

4. Create a connection in AWS CodePipeline
- Go to "Settings" > "Connections" 
- Click "Create connection"
- Select "GitHub" and provide a name and then click "Connect to GitHub"
- Click "Install a new app" and then authenticate to GitHub

# Lab 2 - Launch EC2 instance and install CodeDeploy agent

1. Create an IAM role for EC2 with CodeDeploy and Systems Manager permissions
- Choose EC2 as the trusted entity type
- Add the `AmazonEC2RoleforAWSCodeDeploy` policy
- Add the `AmazonSSMManagedInstanceCore` policy
- Provide a name such as `EC2CodeDeployRole`

2. Launch an EC2 instance 
- Name the instance `EC2CodePipeline`
- Use the Amazon Linux 2023 AMI and a t2.micro instance type
- Enable SSH and HTTP from anywhere
- Launch in a public subnet with a public IPv4 address
- Attach the `EC2CodeDeployRole`role

# Lab 3 - Create a CodeDeploy application and a CodePipeline

1. First create a role for CodeDeploy
- Choose CodeDeploy as the use case
- The `AWSCodeDeployRole` managed policy should be automatically attached
- Provide a name such as `AWSCodeDeployRole`

2. Create the application in CodeDeploy
- Create an application named `CodeDeployEC2App`
- Choose the EC2/on-premises compute platform

3. Create a deployment group in CodeDeploy
- Create a deployment group named `CodeDeployEC2Deployment`
- Select the `AWSCodeDeployRole` role
- Choose an in-place deployment type
- Select "Amazon EC2 Instances"
- Enter the Name/value tag corresponding to the EC2 instance
- For agent configuration with Systems Manager choose "Now and schedule updates"
- For deployment configuration, select `CodeDeployDefault.OneAtaTime`
- Disable load balancing

4. Create a CodePipeline pipeline
- Select to build a custom pipeline
- Name the pipeline `EC2ApachePipeline`
- Allow the creation of a new service role
- Select "AWS CodeCommit" or "GitHub (via GitHub App)" for the service provider and select the repo and main/master branch
- Skip the build stage
- For the deployment stage select CodeDeploy and choose the app/deployment group

***check the pipeline runs successfully, you should be able to view the success page using the EC2 public IP***

# Lab 4 - Add a build stage that checks for a dependency

1. Create a service role for CodeBuild
- Choose CodeBuild as the use case
- Name the role `CodeBuildServiceRole`
- No need to attach any policies

2. Create a build project named `EC2CheckDependency`
- Select CodeCommit or GitHub as the source provider and choose the repo and master branch
- Select the Amazon Linux 2023 managed image and choose the standard runtime and latest x86_64 image
- Select the role created earlier (CodeBuild will add the necessary permissions)
- Select "Use a buildspec file" but do not enter the name
- Specify a CloudWatch logs group name - create the log group and make sure the service role has permissions

3. Add a build stage to the pipeline
- Edit the pipeline and add a stage between the source and deploy stages
- Specify CodeBuild and choose the build project
- Use SourceArtifact as the "Input artifacts" and no need to specify and output artifact

4. Add the buildspec.yml file to the project directory
- The file checks for the existence of a file named `dependency.txt` in the root of the project directory (which currently does not exist)
- Commit the changes to the repo and push to CodeCommit or GitHub
- After a few minutes the build should fail with the output "File does not exist, build fails."
- Create the dependency.txt file in the project directory
- Commit the changes to the repo and push to CodeCommit or GitHub
- After a few minutes the build should succeed with the output "File exists, build succeeds."

