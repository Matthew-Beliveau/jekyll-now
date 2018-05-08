---
layout: post
title: Installing Maven
---

Sometimes I have trouble with using:

`sudo yum install maven`

and it wouldn't either install correctly or install the wrong version of Maven. I usally only have this problem on my VMs that run CentOS.
So I am going to keep a recording of how I install Maven here for me in the future when I inevitably forget.

# Making sure Java is installed correctly 

I like to _really_ make sure Java is installed correctly. What this means is I install everything Java jdk related for my purposes.

`sudo yum install java-1.8.0-openjdk* -y` 

This will install anything that openjdk has to offer. Just to make sure you really got it done. 

Then what I like to do is make sure I have the enviroment variable $JAVA_HOME set up because some programs use it and assume that it is already set.
First you're gonna need to edit your ~/.bashrc file. If you don't have one then make it with the touch command. Once you are sure that you have
the file add these to the end of the file:

`JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk`

`export PATH=$JAVA_HOME/bin:$PATH`

`. ~/.bashrc` 

Then run `echo $JAVA_HOME` to make sure the that it worked. 

Boom,  there you go. Java should be installed correctly.

# Installing Maven 

Go to /etc/profile.d directory to create a new config file
`cd /etc/profile.d

sudo vim maven.sh`

Then paste this:

`# Apache Maven Environment Variables

# MAVEN_HOME for Maven 1 - M2_HOME for Maven 2

export M2_HOME=/usr/local/src/apache-maven

export PATH=${M2_HOME}/bin:${PATH}`

Now you have to make maven.sh executable then source the file:

`sudo chmod +x maven.sh

. /etc/profile.d/maven.sh'

Test that it is running correctly by running `mvn -version` 
