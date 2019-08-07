---
layout: post
title:  "How to make a command line tool in Node.js"
date:   2016-10-04 19:45:31 +0530
categories: Node
---

Getting Started
=

The day has finally come. You aren't content to just get things done in the command line; you want to get things done by using badass rockstar tech. Enter node.js. 

A few weeks ago I decided to build a command line that could take a directory of .js files and add an index.html and a running mocha testing environment. Building such a command line tool with node is surprisingly easy, even with vanilla node. 

Prerequisites for this are node and npm, the package manager for node, which contains literally gazillions of modules. You can pick both of these up at the [node.js website](https://nodejs.org/en/download/). 

Create A Package
=

First, create a package of your own by running npm init in the directory where your index.js lives. Click yes through all the options and you'll have a package.json file. This file serves many purposes, but for our purposes it will be for distribution, should you choose, and for making the command available to the terminal. Full documentation on package.json files can be found [here](https://docs.npmjs.com/files/package.json). 

The modification you will make the package.json file is to add the following, where myapp is the command you want to run in terminal, and mypath is the path to the .js file where you will write your script: 

{% highlight javascript %}
{ "bin" : { "myapp" : "/mypath" } }
{% endhighlight %}

According to the docs, bin works by creating a [symlink](https://kb.iu.edu/d/abbe)-- which is like an alias in unix operating systems-- to the executed file in mypath. The package.json is read on installation of the package (when running npm install -g myfile.js), and the location of this symlink will be available in your local directory or globally, depending on whether the install of the package is global or local. 

A symlink is apparently a pretty empty construct, just a pointer really. An alias alone probably won't do us much good, because once the terminal tells the system to try to open the file it won't know how to run it. So there is one more step to making it executable. You will have to add a shebang inside the script file, which directs the system what environment the code should be run in. This should go on the very first line of the script: 

{% highlight javascript %}
 #!/usr/bin/env node
{% endhighlight %}

Caveat
=
The symlink just created doesn't point to what you are working on anymore. It points to the installed global module. In order to test it while you are developing there is another command, npm link. This will ensure that the system-wide symlink points to the file you are actually working on. I haven't actually tried using this, but it seems to be a very important component of correct workflow.

Load Modules
=

You will need to load the required modules. A few modules come preloaded and globally available, and for my project, this was all that was necessary. fs is for managing the filesystem, and child_process is for creating new processes such as opening the web browser on the newly created specRunner file:

{% highlight javascript %}
var fs = require('fs');
var spawn = require('child_process').spawn;
{% endhighlight %}

NPM is the largest open source code repository in the world, and has modules for everything. With a little searching you could find modules to simplify the tasks in my script: file path management, recursing through file structures, and argument parsing, to name a few. In my project I just went ahead and coded them, but it's important to know that robust solutions can be found quite easily. 

Access the Arguments
=
To get arguments into the tool there are two ways. The hard way (which I did in my project, not knowing any better), and the easy way. The easy way is [commander](https://www.npmjs.com/package/commander). It's usage is described in detail [here](https://developer.atlassian.com/blog/2015/11/scripting-with-node/) and [here](https://medium.freecodecamp.com/writing-command-line-applications-in-nodejs-2cf8327eee2), so I won't go into it. (The last link also explores how to take terminal input using [co-prompt](https://www.npmjs.com/package/co-prompt), and yet more resources for sprucing up the interface are discussed [here](https://www.sitepoint.com/javascript-command-line-interface-cli-node-js/).)

To do things the hard way you will need to access the arguments vector, which is an array passed in at invocation time from the terminal. By default it contains two initial arguments, node itself and the path to the file that is running. So we slice: 

{% highlight javascript %}
var args = process.argv.slice(2);
{% endhighlight %}

This will give you the arguments to the file, which you then parse as you wish. 

Call Asynchronous Processes
=

Most of my script turned out to argument parsing, but the last and most important thing to do is to write the files. This is done using the filesystem or [fs](https://nodejs.org/api/fs.html) module which comes prepackaged with node. 

Here's how the specRunner file is written: 

{% highlight javascript %}

function writeSpecFile(){
  fs.writeFile(settings.creationDirectory + settings.specFileName, settings.specString, function (err) {
    if (err) {
      console.log(err);
    } else {
      console.log("The file" + settings.specFileName + " was saved!");
      spawn('open', [settings.directory + settings.specFileName]); //!not opening
    }
  });

}
{% endhighlight %}

You'll also see here that we spawn the new process, via the spawn function, with an argument of 'open'. This is so that the browser automatically opens the specRunner file. It is implemented with the [child_process module](https://nodejs.org/api/child_process.html). Writing the index looks the same: 

{% highlight javascript %}
function writeIndex(){
  fs.writeFile(settings.creationDirectory + settings.htmlFileName, settings.htmlString, function (err) {
    if (err) {
      console.log(err);
    } else {
      console.log("The file" + settings.htmlFileName + " was saved!");
      spawn('open', [settings.creationDirectory + settings.htmlFileName]);
    }
  });
}
{% endhighlight %}

The fs module also allows deletion, which we do when the input command is '-rm'. 

{% highlight javascript %}
targets.forEach(function(path){
    fs.unlink(path, function(err) {               
      if (err) {
        console.log(err);
      } else {
        console.log(`${nameFromPath(path)} deleted successfully!`);
      }
    });
  });
{% endhighlight %}

These are all asynchronous processes, and since they are executed last here it wasn't too important for control flow. 

Conclusion
=
The opportunities for coding scripts in node are vast. It may be the most accessible scripting environment in existence, because the support provided by NPM is so extensive and because the setup process is so easy. 

note: the source code my repository can be found [here](https://github.com/justin-roche/breakglass). 




