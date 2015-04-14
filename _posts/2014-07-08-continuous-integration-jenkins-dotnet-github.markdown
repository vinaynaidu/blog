---
layout:     post
title:      "Continuous integration for .Net projects"
subtitle:   "Setup your own continuous integration server for .Net projects with Jenkins, Github, and MbUnit"
date:       2014-07-08 12:00:00
author:     "Vinay"
header-img: "img/post-bg-03.jpg"
---

# Continuous integration

This post details the steps required to setup Jenkins to download a .Net project from github, build, run tests using Gallio MbUnit, create build artifacts and aggregate results. When I started out with the task at HealthUnlocked, there were plenty articles about jenkins, and github, and .Net, but no comprehensive articles that detailed how to combine all these together.  

On this blog, I've recorded the pitfalls of setting up the server, common errors that I faced, workarounds, and tips and tricks to make it all nice and shiny - and ultimately more useful, in the hopes that it will be useful to me in the future, or anyone else looking for the same things.  


## Setup required software

Assuming the server is a clean slate server running just the OS, I've listed the softwares that need to be installed. If something is already available, it can be skipped, of course.

* <a href="https://jenkins-ci.org/" target="_blank">Jenkins</a> - The workhorse that is a continuous integration server
* <a href="https://java.com/en/download/" target="_blank">Java</a> - Jenkins relies on a JVM behind the scenes
* <a href="https://www.visualstudio.com/" target="_blank">Visual studio</a> - VS is not strictly necessary, however .Net Framework is. But Once you have a framework, there are plenty configurations and settings you will need to tweak and installing an express edition of VS will take care of most of that and leave your hands free to work on other things
* <a href="http://git-scm.com/downloads">git</a> - Since we'll be pulling our source code from github, we'll need git commands to be available on the environment variables

Once all the required softwares are installed, we're well on our way to start configuring the job. Jenkins, after installation, makes itself available on the port 8080 so it can be accessed by pointing the browser to `http://localhost:8080/`
