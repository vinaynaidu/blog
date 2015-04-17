---
layout:     post
title:      "Continuous integration for .Net projects"
subtitle:   "Setup your own continuous integration server for .Net projects with Jenkins, Github, and MbUnit"
date:       2014-07-08 12:00:00
author:     "Vinay"
header-img: "img/post-bg-04.jpg"
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

Right. That's our brand new jenkins server running. But it needs to understand how to work with git, github, and other parts of our build workflow. This is done by installing plugins. I've listed the required plugins, plus some of the plugins I like to use to make it all more useful. You can pick which plugins you want to install.

To install the plugins on jenkins, open up the web interface and go to `Manage Jenkins > Manage plugins` and switch to `Available` tabs. From here on, you can search the plugins, enable the ones you want, and install them all at once.

* <a href="http://wiki.jenkins-ci.org/display/JENKINS/Github+Plugin" target="_blank">Github plugin</a> - This plugin integrates Jenkins with github and makes it very easy to configure builds and set status on commits and pull requests
* <a href="https://wiki.jenkins-ci.org/display/JENKINS/MSBuild+Plugin" target="_blank">MSBuild plugin</a> - This allows you to use MSBuild console command to build .NET projects
* <a href="https://wiki.jenkins-ci.org/display/JENKINS/Gallio+Plugin" target="_blank">Gallio plugin</a> - Makes it possible to publish the test results.

Later in the post, I will go through the optional plugins that I like to use and what they do.

## Set up Github repository

Now that we have all the setup work out of the way, lets start with configuring a new job! Hit the `New Item` from the main menu on Jenkins web interface, and choose a **Freestyle project** - give an appropriate name to your project and you're ready to configure a freestyle, mix and match style job. 

There are several ways you can configure your job to pull from Github. The best (read most secure, and, recommended) way, is to use a deploy key. <a href="https://developer.github.com/guides/managing-deploy-keys/" target="_blank">Deploy keys</a> are unique to each repository and will grant the same permissions to anyone using the deploy keys the same permissions the user creating the deploy key has (which is also configurable). So, when you're creating the deploy key, make sure you have to right read and write permissions.

We will now need to configure git on the server to work with the deploy keys. Following steps detail how to do that: 

* Create ssh keys with `ssk-keygen -t rsa`
* Create a config file at `~/.ssh/config` with the content:

		Host {Project-name}
			Hostname github.com
			User git
			IdentityFile /path/to/id_rsa

* Go to `github.com/{Project-name}/settings/keys` and use the `id_rsa.pub` created on step 1 to generate your new deploy key.

Now, back in jenkins web interface, on the new job that we just created, select **Git** under *Source Code Management* with the following values: 

* Repository URL: `git@github.com:OrganizationName/Project-name.git` - this *has** to be in the same format. If the project is under a username rather than an organization, replace OrganizationName with the username.
* Credentials: Click on `Add` then: 
	* select **SSH Username with private key** for `Kind`
	* Fill in the server/windows account username for `Username`
	* Description is optional and can be empty
	* For `Private key`, select **From Jenkins master ~/.ssh**

And then hit Add. If everything went well, we should now be able to download the code from Github Repo by building the job.

<small>*Troubleshooting hint:* If there is a problem with the timeouts, it us usually the case of not having the right permission to pull the code. This can be debugged by opening the command line as the user Jenkins is running under, and manually pulling the code with `git pull project/repo/on/github.git` and that should output any errors that are in he configuration</small>

Then, under `Build Triggers`, choose **Build when a change is pushed to github`. This will build the project everything there has been a new commit on Github.

## Building the project

We have our job downloading the code from Github. Let's go ahead and build it. Remember the MSBuild plugin we installed above? It needs to be configured to work with the msbuild.exe on the server. Go to `Manage Jenkins > Configure system` and locate the **MSBuild** section. Hit the **Add MSBuild** button and configure it with the following values: 

* Name: `MSBuild4` - This can be anything, I usually name it after the .net framework
* Path to MSBuild: `c:\Windows\Microsoft.NET\Framework64\v4.0.30319\msbuild.exe` - this is the default path of the msbuild.exe, though it may defer depending on the configuration.
* Default parameters: leave this field empty
* Install automatically: leave unchecked

Then hit save.

<center>![]({{ site.baseurl }}/img/msbuild-warning.png)</center>

<small>*Note:* Once configured, Jenkins may warn about `Path to MSBuild` is not a valid value, something along the lines of **`c:\...\msbuild.exe` is not a directory on the Jenkins master** - this can be ignored safely. I am note entirely sure why the warning popups up, but if I remove `msbuild.exe` and just list the parent directory, the build fails.</small>

Back in the main job configuration, Add a new build step - **Build a visual Studio project or solution using MSBuild** with the configuration: 

* MSBuild Version: **MSBuild4** - choose the name of the build we just setup earlier
* MSBuild Build file: **Project.sln** - the solution name or project name of the project that needs to be built.
* Command Line Arguments: Arguments to be passed to `msbuild.exe`

		/p:Configuration=Release_MySQL /p:DeployOnBuild=true /p:DeleteExistingFiles=True /p:PublishMethod=FileSystem /p:publishUrl=C:\Publish\Aurora-Api
 

<small>*Note:* `/m:2` on command line arguments will enable parallel builds, while visual studio version can be changed with `/p:`</small>

I like to make sure everything is working after each step, by building the project to see if it's doing what it is supposed to be doing. This lets me fix any issues that may crawl up before it bubbles up to the further steps

## Running unit tests with Gallio MbUnit

Under build step, add a new `build step > Execute windows batch command` and add the command line to run gallio.echo.exe on the project.

	"C:\Program Files\Gallio\bin\Gallio.Echo.exe" /report-type:xml /verbosity:quiet "Project-name.Tests\bin\Release\*.Tests.dll"

The command can just be run on command line, from the workspace folder, to make sure it works and tweak if necesary before adding it here.   

Once that's done, Jenkins should be fetching code from github, building and testing the solution merrily. Hooray!

<center>![]({{ site.baseurl }}/img/jenkins-build-and-test.gif)</center>

The tests are being published to `Reports` folder though, it would be more useful if it was displayed with the job, and we could see an overview of what failed. Remember the Gallio plugin we installed earlier? that's exactly what the plugin does. So let's go ahead and configure that.

Create a new **Post Build** step and add the `xUnit test result report` step

* Choose **Gallio-N/A** 
* For Patterns, add `Reports/*.xml` as that's the location of our test report xml output (configured on command line step above)
* Check **Delete temporart JUnit files** and **Stop and set the build to failed status if there are errors when processing a result file**. Depending on the project type, the others can be enabled too. 

You can also configure the failed and skipped test threshold limits. Leaving the boxes empty will fail the build if any test fails or is skipped. 

Once configured, the section will look something like this

<center>![]({{ site.baseurl }}/img/publish-report.png)</center>

Simple stuff!

## Configuring useful post build tasks

While the above steps are sufficient to build and test the project, we can do improve the job and make Jenkins more useful. 

#### 1. Archive the artifacts

Artifacts are the published files - the resulting files of a successful build. We can tell Jenkins what needs to be archived so the archive can be directly published to staging server, if needed, or downloaded and used on whim. 

Set the artifacts by adding a new `Post Build > Archive the artifacts` task and in the **Files to archive:** section, provide a csv of all the necessary files. Wildcards can be used as well.

	Project-name\bin\*,Project-name\Global.asax,Project-name\lib\*,Project-name\Web.config,Project-name\db.config,Project-name\settings.config,Project-name\log4net.config,assets\*

If the project is hosted on EC2, there are AWS plugins available that will enable the artifacts to be deployed directly to the EC2 instance.

#### 2. Scan workspace for open tasks

Visual studio has a neat feature where all the comments starting with `//TODO:` are listed under the **TODO** pane. We can use a plugin in Jenkins that scans all the files and keeps track of such comments, and lists new items or graphs count over time. I find it useful to spreed through them to see if there is something important that needs to be done before a release

The functionality is available via the <a href="https://wiki.jenkins-ci.org/display/JENKINS/Task+Scanner+Plugin" target="_blank">Scan workspace plugin</a> and can be included in the job by adding a new post build step. I add the step **Scan workspace for open tasks** and configure it like so: 

* Files to scan: `**/*.cs` - this scans all directores in the workspace for .cs files
* Files to exclude: leave blank
* Tasks tags: **High priority:** `Fixme`, **Normal:** `TODO`, **Low:** `TBD`

That will now help Jenkins keep tracks of the list of things for us 'to do'

#### 3. Workspace cleanup after successful build

The Gallio plugin will scan `Reports/*.xml` for all reports and each time the test is run, that will result in a new xml file. If the directory is not clearned, they accumulate over time and when one of the test fail, they always fail - because each time the plugin looks at all the files. This was one of the gotchas that took me a while to uncover in the very beginning. There are several ways we can avoid it, but the easiest would be to cleanup directories after the build. The results will be stored in Jenkins db so removing the reports after they are archived will not affect the tracking capabilities of Jenkins.

<a href="https://wiki.jenkins-ci.org/display/JENKINS/Workspace+Cleanup+Plugin" target="_blank">Workspace cleanup plugin</a> provides a minimal configuration and is very easy to configure. 

Add this step to the job with, yet again, new post build task **Delete workspace when build is done** and set the configurations as needed:  

`include (from dropdown)` and `Reports/*` tells it to delete all filese from our reports folder

When that's done, you can see jenkins scanning for the comments 


<center>![]({{ site.baseurl }}/img/jenkins-post-build.gif)</center>

#### 4. List github branches with build status

Jenkins can be configured to build specific branches only, but if you're building all branches then the list of successful builds on Jenkins can get confusing. There is no easy way to say which build was from which branch, short of clicking each of them and checking for the commit hash and comparing tasks on github.

<a href="http://wiki.jenkins-ci.org/display/JENKINS/Groovy+Postbuild+Plugin" target="_blank">Groovy post build plugin</a> will help us with adding custom values to the build list. 

On post build, add a new **Groovy Postbuild** task, and enter the following script in the script textbox: 

	def matcher = manager.getLogMatcher(".*Checking out Revision (.*) (.*)\$")
	if(matcher?.matches()) {
	    branch = matcher.group(2).substring(8,matcher.group(2).length()-1)
	    commit = matcher.group(1).substring(0,6)
	    githuburl = manager.build.getParent().getProperty("com.coravy.hudson.plugins.github.GithubProjectProperty").getProjectUrl().commitId(matcher.group(1))
	    description = "<a href='${githuburl}'>${commit}</a>"+" - "+branch 
	    manager.build.setDescription(description)
	}

And now, all our builds list the branch name along with the status! 

<center>![]({{ site.baseurl }}/img/build-history.png)</center>

#### 4. Github status update

<a href="https://developer.github.com/v3/repos/statuses/" target="_blank">Github statuses</a> are awesome. It lets you know which commits are broken and which ones are great before you merge the pull request so you know you don't need to worry about build failing after merge if you're only building master branch. 

The Github plugin we installed earlier takes care of it. All we need to do to update the build status on github is add a new Post build task **Set pending status on github** and **Set build status on github** at the beginning of the build, and at the end, respectively. 

<center>![]({{ site.baseurl }}/img/jenkins-github-status.png)</center>

---

Jenkins has plenty of other integrations as well. At HealthUnlocked, we're a big fan of <a href="https://slack.com/" target="_blank">Slack</a> and we have a channel of build and test notifications. Slack makes it very easy to write integrations and API's that can post to channels and we have our Jenkins posting the build status so we know immediately if a build fails! 

<center>![]({{ site.baseurl }}/img/jenkins-slack.png)</center>

All these extra (minimal effort) measures will make sure that not only the code everyone writes is well and good, but also notifies people immediately when something is wrong, which is very important, as it eliminates the "out of sight, out of mind" scenarios. Being able to act on a problem is well and good, but being able to know about a problem is just as important.


## Troubleshooting

I thought I'd add a section about the common problems I've faced, while installing Jenkins several times. More often than not, I'd see the same errors crop up and it was handy to have a note of what I did fix it last time it happened.

#### Nuget package manager fails to download packages

While compiling from within the Visual Studio IDE, the user has permissions to automatically pull in the depending packages. This isn't the case when we're building the project using msbuild.exe and Nuget will not have the right permissions to download. This is an easy to fix problem, though, luckily - Add a new Environment variable `EnableNuGetPackageRestore` with value **true** under system variables and that will take care of this problem.

#### Error MSB4019: Microsoft.WebApplications.Targets was not found

This is an annoying problem to be stuck with and it took me a while to work out what was happening. Copy over the `WebApplications` folder from your dev machine to the build server and that will fix the problem. The file is usually located at 

	C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v10.0\WebApplications

You can find out more about this problem at <a href="http://stackoverflow.com/a/5344246" target="_blank">this stackoverflow answer</a>.

#### Error CS0234: The type or namespace name 'Mvc' does not exist in the namespace 'System.Web'

What? I swear I installed the right MVC framework on my build server! Well, turns out the build number mismatch causes this problem as well. You could try copying the dlls found on the dev machine over to the build server, but, that will not always work and what's more, we'll need to repeat this every time we steup a server.

Better way to do this would be to remove existing MVC reference on the project, and install it from NuGet. That will help jenkins just download the same version of the package.
