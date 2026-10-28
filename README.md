# 100 Days of DevOps with KodeKloud - Days 71 - 80

We finished up Kubernetes so now the next 10 days are all about Jenkins! Lets learn something new. 

## Day 80: Jenkins Chained Builds



## Day 79: Jenkins Deployment Job



## Day 78: Jenkins Conditional Pipeline



## Day 77: Jenkins Deploy Pipeline

Okay this part I'm more familiar with on the Jenkins side. I've created a few pipelines but never from scratch so I still need to memorize the syntax to a certain degree. Before we even get started, I installed the following plugins: Publish Over SSH, SSH Build Agent, Pipeline, and Matrix Authorization Strategy. I figured I would need majority of these. After that, I created the SSH server session under Systems for Application Server 01 `stapp01` and verified connectivity. I had to create the App Server 1 agent next with the root directory being `/home/sarah/jenkins_agent`. This caused me about 20 minutes worth of issues and I didn't know how to fix it. I'm also very sleepy. I asked AI what the issue was. There were a few differeht issues. #1 was the credentials we used for the SSH session under the Agent tab. We needed Tony's credentials instead of Sarah's which I figured at first. AI was wrong initially here. I also needed to change the launch credentials for the agent from Host Key Verification Strategy → Non verifying Verification Strategy. Then, AI pointed out Tony doesn't have rw permissions on the directory `/home/sarah/jenkins_agent` so we needed to run:

```
sudo mkdir -p /home/sarah/jenkins_agent
sudo chown tony:tony /home/sarah/jenkins_agent
sudo chown -R tony:tony /var/www/html
```
```
sudo setfacl -m u:tony:x /home/sarah
sudo setfacl -m u:tony:rwx /home/sarah/jenkins_agent
```

just to give Tony some type of permissions on the directory while keeping Sarah as the owner and group for it. Then, the last issue was `stapp01` didn't have the correct Java version. We needed to run `sudo dnf install -y java-17-openjdk` to upgrade it to the correct version. Finally the agent came online. What a headache. Lets run the Jenkins pipeline and see if it completes the lab. 

```
pipeline {
    agent {
        label 'stapp01'
    }

    stages {
        stage('Deploy') {
            steps {
                sh '''
                    cd /var/www/html
                    git config --global --add safe.directory /var/www/html
                    git pull
                '''
            }
        }
    }
}
```

The job ran successfully after this and we were able to get the green check. This took 50 minutes believe it or not. I learned a lot though. 




## Day 76: Jenkins Project Security

This lab was also pretty easy. After doing a bunch of aimless clicking, I googled the part of the prompt that said inherit permissions from parent ACL. This pointed me back towards the Matrix Authorization Strategy plugin. Once I installed that, I did more ClickOps but couldn't find what I was looking for. AI pointed me back to the job and said I needed to check Enable project-based security. Once I did that, I was able to add the `sam` and `roahn` users and give them the required permissions for this job. 

## Day 75: Jenkins Agent Nodes

This lab was a lot longer than I exected. We needed to create 3 agent nodes for the application servers. Before we did that, we needed to make sure the agents would come online by creating SSH credentials for them. So I needed to install SSH Build Agents (not SSH Agents) and Publis Over SSH plugins. Afterwards, I created SSH servers for each of the agents and tested my configuration. Next, I created agents for all three of the servers as specified in the lab requirements. They still didn't come online so I needed to go back into the agent configurations and change the option to `Launch by SSH` or something simliar to that. This prompted me to choose a server and then remake my credentials in the form of a Jenkins Credential for all 3 servers. After that, the agents were online one of them kept going offline. I looked at the logs and it appeared that the SSH session was still timing out. 

I still hit enter and got the green check though. Mission accomplished. 

## Day 74: Jenkins Database Backup Job

I knocked this out very swiftly on my own. So we were given the credentials and information for the database we needed to back up every ten minutes. We needed to configure a Jenkins pipeline to automate this process. I found this command online: `mysqldump -u [username] -p"[password]" [database_name] > [output_file].sql` that we can use in the future for the actual database dump job. We also needed this scp command `scp db_$(date +%F).sql natasha@ststor01:/home/natasha/db_backups/`. Everything else is straightforward. Just make sure you install Publish Over SSH first, test the connection to the machine you'll be starting from, and run the SSH commands from the previous lab (Day 73) from the source machine to the destination machine to make sure we don't run into that fingerprint and password prompt. Those two above commands should create the cron job. 

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
