---
layout:     post
title:      "Customizing fiddler"
subtitle:   "Start fiddler with composer window seperated (and at a fixed position)"
date:       2015-08-27 18:34:00
author:     "Vinay"
header-img: "img/post-bg-fiddler.jpg"
---

I use <A href="http://www.telerik.com/fiddler" target="_blank">fiddler</a> for all my api testing, development and debugging purposes. 
It's simple enough to start with and comes with a lot of functionalities. It lets me capture incoming and outgoing traffic from any application
and inspect headers and response in a range of formats. All in all, it's a great tool in any developer's arsenal and over the years I've come to 
love it.  
   
One thing that always annoyed me, though, is that fiddler starts up like this:  

<center>![]({{ site.baseurl }}/img/fiddler-old-startup.png)</center>  

Being primarily interested in testing out my REST API, every time I start up fiddler, I: 
    * switch to `Composer` tab,then to `Options` and check the `tear apart` option, to seperate the composer window
    * Move and resize seperated window to appropriate position 
    
This looks something like this: 

<center>![]({{ site.baseurl }}/img/fiddler-composer-away.jpg)</center>
    
This lets me easily test my api endpoints and inspect the response without a lot of switching tabs around. 
While the main window remembers it's position and size, the composer window starts of fresh and I always have to manually fix it. Over the last 4 or so years, 
it's become a bit of a chore to manually doing this and I while wondering if there was a way to better way to do this, 
I shot off an email via fiddler feedback (under help menu) describing my (trivial) problem.  

I got a reply right away from the awesome <a href="https://twitter.com/ericlaw" target="_blank">Eric Lawrence</a>, the creator of fiddler, detailing 
how to do just that. Turns out, fiddler supports customisations through scripting and to automate what I want, all I had to do was:  

    * Click Rules > Customize Rules to open the FiddlerScript
    * Scroll to the `OnBoot` function and add the following lines inside it: 
      
    FiddlerApplication.UI.actQuickExec("!composer move 123,456,789,1000");
    FiddlerApplication.UI.actQuickExec("!composer activate Scratchpad");
    
The first line get composer to detach and itself from main window and set it's position on screen, with `XPos`, `YPos`, `Width` and `Height` for the composer's window.
The second line will open up the `Scratchpad` tab on composer window. You can of course change it to `Parsed` or any other tabs and that works well enough.  

Things to note here though, initially, my whole `onBoot` method was commented out, and I had to uncomment it to get this working. Oh, the 
FiddlerScript is supported from version `4.4.7.1` and up.

Another cool tip about fiddler: if you usually don't want to capture all the traffic going in and out from every application, but just want fiddler's traffic, 
start fiddler with `-noattach` parameter and it will do just that. I've added shortcut to my taskbar with this param and use that when I'm working with
endpoints, which is most of the time!  