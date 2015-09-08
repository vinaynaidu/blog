---
layout:     post
title:      "Kickass custom Slack bot"
subtitle:   "Building a custom bot that responds to slash commands using slack integration using clojure"
date:       2015-09-08 15:16:00
author:     "Vinay"
header-img: "img/post-bg-slackbot.jpg"
---

<a href="https://slack.com" target="_blank">Slack</a> is cool. I've used it for quite a while now and I think it's my favourite communications tool so far. What really makes slack amazing is its amazing integrations. There's an integration for pretty much everything you can think of. Github notifications, jenkin builds, jira task managements, my favourite <a href="" target="_blank">random gifs</a> for reactions and responses... and the best one by far - custom integrations.  

What's so great about custom integrations? You want to push certain version of your app to staging or test machine? just type `/jarvis deploy api to test`, or maybe `/jarvis deploy pacman 2.3 to staging`. Or maybe you just want a definition of some big word someone just used; type in `/jarvis define retrospective`. How about a fun random definition of something from urban dictionary? you can do `/jarvis urban senator` or even `/jarvis urban IIRC` if you cannot recall the full form of an acronym from the ever expanding list. Oh, I know, how about this - you're having a bad day and you just need something to cheer you up. Jarvis to the rescue again! Type `/jarvis pugme` would result in the bot posting a random picture of a pug (courtesy of <a href="https://pugme.herokuapp.com" target="_blank">pugme</a>). Anyway, you get the idea.   

I setup a custom slash command backed by a bot running on Heroku, because, hey, heroku is awesome and free. The command, lets assume, was called `Jarvis` (yes, borrowed from the famous Ironman). Jarvis let me do all of that, and has plenty room to extend the supported commands.  
   
I'll go over the steps I took to build the slack bot, and mention any reasons that went behind the choices. Word of warning though, this is going to be a high level overview of the process, rather than a step by step walkthrough. Nevertheless, there will be enough details to put the pieces together to make your own slackbot.   

### Slack integrations

First things first, you will need a slack account (I shouldn't really have to say this!). Slack is amazingly developer friendly and support <a href="https://api.slack.com/" target="_blank">multiple types of integrations</a>. Incoming and outgoing webhooks, Realtime Web Api, custom slash commands, just to mention a few. Their documentation is very detailed and the support team are very friendly too, I was stuck around a few minor things a couple of times and both time they helped me understand the api's and possible workarounds for what I wanted to achieve, when it wasn't directly supproted. So all in all, I'd say if you want to develop a custom slack integration, you're in good hands.  

The easiest choice to work with, that takes minimal effort to setup, is the remote control. It is literally a `HTTP POST`. No fuss. The next level up is webhooks, and naturally I went with the incoming and outgoing webhooks at first. But I soon realized that they were very restrictive in terms of permissions. I wanted the bot to respond to calls anywhere - private and public channels, or one to one messages; and it's something you cannot do with either webhooks or remote control. So the current bot works with a slash command.  

The slash command request also expects a 200 header response and is limited in formatting options. I had to mention who issued the command etc and the incoming webhook has great formatting options for that kind of thing, so almost all tasks, as soon as they recieve the slash command, send an empty 200 response and then follow up with an incoming webhook with actual response. For the end user, the difference is nil, all they see is they get a response for their slash command, they are none the wiser and I get to process and format my commands. It's almost as good as getting a cake AND having it.   

### Setup a custom slash command

The idea behind slash command is simple enough - you setup a <a href="https://api.slack.com/slash-commands" target="_blank">new slash command</a> and every time someone issues a the slash command, some data is sent to configurable URL via `HTTP POST`. Taken from the documentation, the `POST` data could look like this:  

    {
        token=gIkuvaNzQIHg97ATvDxqgjtO
        team_id=T0001
        team_domain=example
        channel_id=C2147483705
        channel_name=test
        user_id=U2147483697
        user_name=Steve
        command=/weather
        text=94070
    }

Public channels are identified by `channel_id`, private channels are identified by `private_channel_id`, and DM's (direct messages) are identified by `user_id`. However, `channel_id` property is named ambiguously, and will contain `private_channel_id` or `user_id` when posted from private channel and DM respectively. Simply put, we can always send the response with original `channel_id` to post to the same place the slash command was issued from.  

The key to posting in any channel - public, private, or DM, is the `channel_id`. Even though it's identified with `user_id`, the `channel_id` contains the right id for any channel or DM.  

Once you follow the brain dead simple instructions to setup a slash command, Slack generates a unique `Token` for that integration. This token will have to be included in any response json that is being sent as a reply.  

<center>![]({{ site.baseurl }}/img/slack-custom-slash-command.jpg)</center>  


### Making of Slack bot - SpaceRain

I used clojure to build my slack bot for two primary reasons:  

 * I was closely involved with clojure at the time of starting this bot
 * Hosting clojure apps with heroku was extremely easy
 * I really, really love clojure. It's a functional programming bliss.

I said two primary reasons... the last one was probably more important in this descision making than the other two!  

The entire project is available on github, under the codename <a href="https://github.com/vinaynaidu/SpaceRain" target="_blank">SpaceRain</a>. I like using codenames for my projects so if the project evolves and deviates from what it originally was planned to do, then the namespaces etc won't be left behind; but it's also a lot of fun to use codenames.  

*Note:* The deployment and rollback part of the code has since been moved to it's own project and and SpaceRain simply delegated the process to that project. As all the servers were run on Amazon EC2, the access keys etc had to be kept seperate and internal and that project was run on an internal machine. For simplicity reasons, SpaceRain contains none of the deployment code.  

Building the actual bot is probably the easiest (and thankfully the most fun) part. The basic idea behind slash command is quite simple.   

 * User issues a slash command, eg, `/Jarvis define protons`
 * Slack does a `HTTP POST` with the above given payload to the custom url that was configured when the slash command was setup. Since my bot was deployed on heroku, the url was `http://spacerain.herokuapp.com`. 
 * `text` parameter in the request will have any text (`string`) that was entered after the command was issued. For eg, the above command would cause `text` to contain `define protons`
 * Read in the `text` parameter, do the necessary steps to generate appropriate response JSON payload and reply back with a 200.
 * Slack expects a 200 JSON response for the request. If the result is anything but a valid 200 Json payload, slash burps the entire error message into the channel so make to put the code in a nice try catch block and issue empty response with a message such as `invalid attempt` or something similar to keep things clean at the user end.
 * the `POST` body will have all the data that is required to process the command and respond to.  
 
And that's it, the entirety of slack bot request life cycle. If multiple slash commands are calling the same url, the `command` parameter is useful to read in the issued command. For my purposes, I just used one slash command, and several sub-commands style, so I wouldn't have to modify several command configurations if there was ever a need to move them around.  

What the backend does is quite straight forward. it reads in the `text` parameter and the first word is always the *sub command*. So, my bot worked based on these parameters:  

| Command | sub command | args | interpretation |  
|---|---|---|---|  
| `/jarvis define osmosis` | `define` | `osmosis` | display meaning of "osmosis" |  
| `/jarvis urban Net Forget` | `urban` | `Net Forget` | Search in Urban dictionary for "FWIW" |  
| `/jarvis pugme` | `pugme` | `nil` | show me a random image of a pug! |
| `/jarvis deploy api to test` | `deploy` | `api to test` | deploy latest build from jenkins to test machine |

This meant that in clojure, all I had to do to work with sub commands, is put the `text` parameter through a <a href="https://github.com/vinaynaidu/SpaceRain/blob/master/src/spacerain/core.clj#L23-L30" target="_blank">switch block</a> and call appropriate function.  

    (let [sub-command (first text)
          args (string/join " " (rest text))]
        (case (first text)
          "pugme" (t/pugbomb 1)
          "define" (t/define args)
          "urban" (t/urban-define args)))

From here on out, the function is responsible for doing the appropriate task and send a json payload response with 200 headers. For example, the `pugbomb`'s task is to get a random pug image and send that url path in the payload, something akin to:   

    (let [pugs (client/get "http://pugme.herokuapp.com/bomb?count=1")
          pug-url (-> (ch/parse-string (get-in response [:body]) true) :pugs first)
          response {:text p
                    :username "Jarvis"
                    :channel channel_id}]
        (post-to-slack response))

Here, I'm using `pugme.herokuapp.com` api to get an array of random pug image urls, with count 1 so there will just be one item in the array, and returning that url in payload.  

All tasks are performed in the `spacerain.tasks` <a href="https://github.com/vinaynaidu/SpaceRain/blob/master/src/spacerain/task.clj" target="_blank">namespace</a>. Have a look around the project and you will understand how most functions work.  

### Mashape API

I mentioned earlier that the bot provides cambridge dictionary and urban dictionary meaning to words, via `define` and `urban` sub commands. I originally looked into using their own api's and while there were plenty free dictionary api's available, they quality was questionable and cambridge only had a pay to use api's. Urban dictionary offered no api's at all. After looking around a little bit more, I ended up using <a href="https://market.mashape.com/explore" target="_blank">Mashape API</a>. It somehow provides a lot of good quality api's, urban dictionary included! and it is free to use upto certain requests an hour. Seeing as the bot was being used for a simple in-house purpose for a small startup, this suited me perfectly.  

You'll need to signup to <a href="https://market.mashape.com/" target="_blank">Mashape</a> and generate your own application key to continue using the api's.  

Another thing to note here: Heroku uses <a href="https://devcenter.heroku.com/articles/config-vars#setting-up-config-vars-for-a-deployed-application" target="_blank">environment variables</a> for configuration, so they'll have to be setup using heroku toolbelt. I'm using <a href="https://github.com/weavejester/environ" target="_blank">Environ</a>, a clojure library to read in my <a href="https://github.com/vinaynaidu/SpaceRain/blob/master/src/spacerain/config.clj#L32" target="_blank">Mashape key</a>  


That pretty much details the SpaceRain project and how I setup a custom Slack bot. Feel free to use the code, fork the repo, and/or submit pull requests.  