---
layout:     post
title:      "Autotest for Go"
subtitle:   "Run your Go tests automatically with the entr(1) unix utility"
date:       2015-01-21 12:00:00
author:     "Brandon Keene"
header-img: "img/post-bg-01.jpg"
---

Running your tests automatically is good. It should be part of your workflow.
It should also be so simple to set up that you actually do it.

The best autotest runner / file watcher tool I've found is 
[entr(1)](http://entrproject.org). ENTR, or the Event Notify Test Runner, is a 
simple UNIX utility that watches your files for changes then re-runs your tests.
ENTR is not Go specific and can be easily used with any language or framework.
It works great with Go because the test run blazingly fast.

It can be easily installed with your favorite package manager:

On Ubuntu:

    $ sudo apt-get install entr

On OSX with Homebrew:

    $ brew install entr

The homepage has great [documentation](http://entrproject.org) so I won't repeat 
too much. Here's a small bash script called `autotest.sh` that I add to my Go
projects:

{% highlight bash %}
#!/bin/bash

which entr > /dev/null
if [ $? -ne 0 ]; then
    # your package manager here
    echo "installing entr with homebrew"
    brew install entr 
else
  echo "go autotest started"
fi

while sleep 1; do
  # add your own test command and flags
  ls *.go | entr -d -c go test ./... -logtostderr=true
done
{% endhighlight %}

I have my editor open side by side with `autotest.sh` and its super 
helpful to have near instant test and compilation results.

Hopefully this speeds up your development and encourages you to write more
tests!

Tags: #golang, #go, #programming, #bash, #protip