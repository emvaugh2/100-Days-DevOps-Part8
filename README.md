# 100 Days of DevOps with KodeKloud - Days 71 - 80

We're now in the 60s! This is still mostly Kubernetes (K8s) labs but we'll end off running some basic Jenkins labs. Lets get it started. 

## Day 80: Jenkins Chained Builds
## Day 79: Jenkins Deployment Job
## Day 78: Jenkins Conditional Pipeline
## Day 77: Jenkins Deploy Pipeline
## Day 76: Jenkins Project Security
## Day 75: Jenkins Slave Nodes
## Day 74: Jenkins Database Backup Job
## Day 73: Jenkins Scheduled Jobs

We need to build a job that copies log files from one server to another every 6 minutes. There's a cron like setting in the Jenkins jobs under Triggers. If you choose `Build periodically`, you can put a cron type of format here to say when to run this job. My format was `H/6 * * * *` which Jenkins will show you when this job will run. It shows every 6 minutes on here so this will suffice. 

UPDATE: I asked AI for feedback on my solution and my design was wrong. I defintiely needed `scp` which is fine but AI wanted to simplify the solution to make the Jenkins server the middle man. So we'll be copying the files from the Application Server to the Jenkkins server. Then, we'll get them from the Jenkins server to the Storage Server. I had to install Publish Over SSH again and put the Application Server (stapp01) info there. 

UPDATE2: We skipped the build step completely and in Environment, we chose `Send files or execute commands over SSH before the build starts`. Then we chose the app-server-1 and gave it the command 

```scp /var/log/httpd/access_log /var/log/httpd/error_log natasha@ststor01:/usr/src/security/``` 

to run. Before we did this, we had to log into `stapp01` to and run the following two commands to bypass the fingerprint and password fields when securely copying to the storage server: `ssh-keygen -t rsa` and `ssh-copy-id natasha@ststor01`. 

## Day 72: Jenkins Parameterized Builds

Okay I think I was able to figure this out by myself. We needed to create a Jenkins job that echos the values of the Stage and env parameters. In the job, I chose the 'This is a parameterized build" and chose to create a string parameter named Stage. I also created a choice parameter named env. I don't know what a choice paramter is. In the build, we needed to echo both of these values so I wrote the `echo "$Stage"` and `echo "$env"` in the Build section. I ran the job and it allowed me to choose my parameters which was mainly for the env variable since that could have the values Development, Staging, and Production. The job was sucessfull and I saw the echo output. Got the green check. 

## Day 71: Configure Jenkins Job for Package Installation

Okay this lab was beyond me. Im used to building pipelines using Groovy. This was a freestyle project. The overall goal was to created a string parameter named PACKAGE, log into the storage service and install the package associated with the string parameter. The value of our parameter is `vim-enhanced`. Now, I learned in freestyle projects, you can type the Bash commands directly in the build steps so Jenkins can run them. The issue was, the job wasn't that simple. I needed to go to the General section in the Configure part of the job, check "This project is parameterized", choose String Parameter, name is PACKAGE, and give it the value vim-enhanced. Afterwards, I had to install the Publish over SSH plugin. Once I installed it, I navigated to Manage Jenkins and clicked on System. Find the add SSH Server section. Here, give your server a name and input the storage server hotname `ststor01` and the user `natasha`. Save your settings. 

UPDATE: There's a Test Configuration button under the SSH Servers section. My test failed so I opened up Advanced and saw there was a box labeled `Use password authentication, or use a different key.` I checked this box and was able to enter a password for the SSH section. I then received the Succeess message. 

In your build job, go to Build Step but this time, choose `Send files or execute commands over SSH` and your storage-server option. In the Exec commands section, put `sudo dnf install -y "$PACKAGE"`. 

The Build came back successfully so I'm going to log into the storage server and see if `vim-enhanced` is installed. You can use the `rpm -q vim-enhanced` command while on the storage server. I see that it's installed so I'll go ahead and hit the Check button. Everything worked!
