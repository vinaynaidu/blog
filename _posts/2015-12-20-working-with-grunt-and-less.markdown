---
layout:     post
title:      "Working with LESS (and Grunt)"
subtitle:   "Tying up LESS with grunt to is more fun than manually pre processing it"
date:       2015-12-20 10:41:00
author:     "Vinay"
header-img: "img/post-bg-grunt.jpg"
---

This blog post is more of a reference for me when I have to setup another JS project with grunt and LESS in the future, so I can pick what's necessary right here rather than googling all over the internet.

The basics of less is covered in great detail on the source website. Less makes it very easy to write component-style css which is easy to maintain and read. Browsers, however, don't understand LESS files natively, so they will have to be processed into `.css` files. Any time a change is made to a `.less` file, it needs to be converted into its css equivalent before the browser can show those changes. This repetitive task can be easily automated with <a href="http://gruntjs.com/" target="_blank">Grunt</a>.

If the current project has grunt already configured, running 'grunt watch' is all that is required to start the process running.

To setup the workflow for new projects, below steps can be used as guideline to configure grunt to automate tasks.


## Configure `grunt-contrib-less` to process `.less` files

* Install the `grunt-contrib-less` package with: `npm install grunt-contrib-less`
* Once the plugin is installed, load the plugin with `grunt.loadNpmTasks('grunt-contrib-less');`
* Add a new task in Gruntfile.js to preprocess less files. The code below instructs grunt to convert `src/styles.less` into `src/styles.css`, while compressing the destination file (specified in the options)

	less: {
			development: {
				options: {
					compress: true
					yuicompress: true
				},
				files: {
					"src/styles.css": "src/styles.less"
				}
			}
		}


* Finally, register the less task in the default task list with: `grunt.registerTask('default', [less]);`


That is all there is to it. Now every time grunt is run, the `.less` file(s) will be processed and converted into css files.



## Configure grunt-contrib-watch plugin to automatically process files on change

Of course, running the grunt task after every change manually can be tedious. Grunt is meant to automate things, afterall, so we can ask grunt to do this automatically as well, by using the `grunt-contrib-watch` plugin. grunt-watch can be told to watch and monitor certain file types (or files) and perform a grunt task when something changes. The tasks can be split into multiple tasks depending on file types or mixed and matched, depending on the project needs. For the purpose of processing less files automatically, calling the above configured less task on file change is enough.

* Install the plugin with npm install grunt-contrib-watch
* Load the plugin by adding grunt.loadNpmTasks('grunt-contrib-less'); into Gruntfile.js
* Add the configurations for the watch task with:

	watch: {
		styles: {
			files: ["src/styles.less"],
			tasks: ['less']
		}
	}

Essentially, this is telling grunt-watch to watch for changes in `["src/styles.less"]` and invoke the `less` task (configured in step 1) anytime the file changes. Now pull up a command prompt, `cd` to project directory, and run grunt watch and the watch task will fire up and start watching the files.

*Note*: It is recommended to not register watch task into the default task list if the CI is configured to run grunt as this will result in files being watched indefinitely, thereby resulting in infinite compile time. Either configure a second task list with 'dev' options and run `grunt dev` for watching if multiple tasks are needed, or just running grunt watch manually.

## Taking watch a step further with LiveReload

<a href="http://livereload.com/" target="_blank">LiveReload</a> is a cool plugin that automatically refreshes the browser anytime a file that it is monitoring changes. What this means, is that it eliminates the need to focus the browser and hit refresh manually every time a change is made. It is a great tool to increase productivity as the programmer can now just focus on the editor and save changes, to see the results automatically update on the browser.
What makes it even better, is that Grunt-watch plugin has livereload built in, all that's needed is to enable it on the watch options configuration. To get the browser to refresh however, there are a few steps required:

* Enable LiveReload in watch configuration

	watch: {
		options: {
			livereload: true
		},
		styles: {
			files: ["src/styles.less"],
			tasks: ['less']
		}
	}


* now running grunt watch will start the LiveReload server on default port 35729.
* Add `<script src="//localhost:35729/livereload.js"></script>` to the index.html file and refresh the page once.

Now the `.less` file is monitored for any change and browser is updated automatically on save. Other files like .html and .js can also be added to the file list so the browser updates when there is a change on those files too.

Since livereload script file is not needed in the index.html on production, grunt can be configured to remove the script on other environmental builds as well. grunt-grep is a plugin that will help with this.

* Install grunt-grep with `npm install grunt-grep`
* Add `<!--@grep dev-->` next to the livereload script tag
* Configure Gruntfile.js with

	{
    grep: {
      options: {
        fileOverride: true
      },
      production: {
        files: {
          'src/index.html': [
            'src/index.html'
          ]
        },
        options: {
          pattern: 'dev'
        }
      }
    }
	}

Finally register the task under default tasks, and you're all set to concentrate on coding something fun without fuffing about with browsers and command lines too often!
