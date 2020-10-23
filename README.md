# sample-spring-boot
Sample Springboot app
“We are planning for the big release of our Hello World Application by end of the day”

## “Task 1: Spin-up our CICD infrastructure”
They want me to get another repo that has their Jenkins in it. They say there might be some configurations I need to fix…

I fork the Jenkins repo to malokingi. I run the k8s commands included. There are a few “errors” according to VSCode in kubernetes.yml. 

The error on the specs part of “PersistentVolumeClaim” says it needs a template and selector. I look up how to do that. I add a selector to it to match the other parts of the file. I can’t find why it’s complaining about the “template” thing, so I move on…

The error on the “Deployment” part said that the spec: selector: matchLabels: is expecting a String, I make it a string. That didn’t help. I suspect, after looking at the other selectors in the file, that we don’t need “matchLabels”. No Errors now, somehow. Oh, the yaml extension for VSCode was just failing for yet unknown reasons. The errors are still there…

I re-apply k8s.yml. It says there were no changes. I need to move on...

## “Task 2: Setup a new multi-branch pipeline job for our hello world app”
I go ahead and make a new pipeline job in Jenkins for https://github.com/Malokingi/sample-spring-boot and set it to trigger automatically when it’s pushed by using ngrok to allow github to get to my local jenkins. 

I add a small shell script to make add, commit, push to git a little easier. GitHub says the payload was sent out okay, but no pipeline triggered.

That’s right, I need to run the pipeline manually first, but that’s not happening due to “waiting to executor” I go to the Master Node and change the number of executors from 0 to 2 :-/ (interesting choice by the last team)

Successful initial running of pipeline. I test the webhook again. Yay it works. 

## “Task 3: Setup sonarqube scan for this project”
I add my github repo to my sonarcloud account. It says none of the languages are supported, I make it analize anyway. I commit as sonarcloud tells me to. No bugs reported. 

Next I download the needed plugins on jenkins. Oh, maybe it’s not a plugin. I follow these instructions. Wait, it’s both. I set those up with a secret text. I look up the command to trigger sonarqube (as opposed to mvn test or whatever I’m used to) and add it to steps under “stage('sonarqube')”. I try “./gradlew sonarqube”. Another test push…
It failed due to permission denied to “gradlew”. I chmod to change access… Wait it’s permissions is already -rwxr-xr-x…

At a loss, I add quotes to the “./gradlew “sonarqube”” and try again… That’s how I got my ./robogit to take params properly, but I thought that was just needed since the param String sometimes has spaces… Let’s see what… It failed again due to “/var/jenkins_home/workspace/sample-spring-boot-pipeline@tmp/durable-3621fcb7/script.sh: line 1: ./gradlew: Permission denied”

I try switching to “./gradlew.bat” Nope. 

Wait the step before does this: “sh 'chmod +x gradlew && ./gradlew build'” could that be messing something up? Or maybe I shouldn’t be using gradlew at all? Since there’s no permission issues with the build step, I try adding “chmod +x gradlew && ” to my sonarqube step and see what happens. 

Well, now it says “gradlew: not found”...

I ran out of time :-(

## “Task 4: Setup docker build and docker push”
I ran out of time :-(

## “Task 5: Setup Application deployment onto your local kubernetes”
I ran out of time :-(

## Postmortem

So, I ran out of time around the end of the SonarQube step. As far as I can tell, everything was fine with Sonarqube itself, it was the gradle-related stuff I was struggling with. I was going to next suspect that the issues were something to do with the:

agent {
    docker { image 'busybox' }
}

part as the step before, which does not fail, has:

agent {
    docker { image 'gradle' }
}

and there was something in the instructions about looking at those Docker Images...


202010231200