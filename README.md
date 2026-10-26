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



## Day 72: Jenkins Parameterized Builds

Okay I think I was able to figure this out by myself. We needed to create a Jenkins job that echos the values of the Stage and env parameters. In the job, I chose the 'This is a parameterized build" and chose to create a string parameter named Stage. I also created a choice parameter named env. I don't know what a choice paramter is. In the build, we needed to echo both of these values so I wrote the `echo "$Stage"` and `echo "$env"` in the Build section. I ran the job and it allowed me to choose my parameters which was mainly for the env variable since that could have the values Development, Staging, and Production. The job was sucessfull and I saw the echo output. Got the green check. 

## Day 71: Configure Jenkins Job for Package Installation

Okay this lab was beyond me. Im used to building pipelines using Groovy. This was a freestyle project. The overall goal was to created a string parameter named PACKAGE, log into the storage service and install the package associated with the string parameter. The value of our parameter is `vim-enhanced`. Now, I learned in freestyle projects, you can type the Bash commands directly in the build steps so Jenkins can run them. The issue was, the job wasn't that simple. I needed to go to the General section in the Configure part of the job, check "This project is parameterized", choose String Parameter, name is PACKAGE, and give it the value vim-enhanced. Afterwards, I had to install the Publish over SSH plugin. Once I installed it, I navigated to Manage Jenkins and clicked on System. Find the add SSH Server section. Here, give your server a name and input the storage server hotname `ststor01` and the user `natasha`. Save your settings. 

UPDATE: There's a Test Configuration button under the SSH Servers section. My test failed so I opened up Advanced and saw there was a box labeled `Use password authentication, or use a different key.` I checked this box and was able to enter a password for the SSH section. I then received the Succeess message. 

In your build job, go to Build Step but this time, choose `Send files or execute commands over SSH` and your storage-server option. In the Exec commands section, put `sudo dnf install -y "$PACKAGE"`. 

The Build came back successfully so I'm going to log into the storage server and see if `vim-enhanced` is installed. You can use the `rpm -q vim-enhanced` command while on the storage server. I see that it's installed so I'll go ahead and hit the Check button. Everything worked!
