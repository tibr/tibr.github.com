---
layout: post
title: Increasing Jenkins Heap Size on Mac OS X
---
When Jenkins is running as a service and you're running into issues because it doesn't have enough heap size, you can increase it by changing the defaults:
{% highlight bash %}
~  sudo defaults write /Library/Preferences/org.jenkins-ci heapSize 2048M
~  sudo defaults write /Library/Preferences/org.jenkins-ci permGen 512M
{% endhighlight %}
These settings get read by `/Library/Application Support/Jenkins/jenkins-runner.sh`.

After restarting Jenkins
{% highlight bash %}
sudo launchctl unload -w /Library/LaunchDaemons/org.jenkins-ci.plist
sudo launchctl load -w /Library/LaunchDaemons/org.jenkins-ci.plist
{% endhighlight %}
you're good to go!

To see a complete list of possible default settings, check out the <a href="https://wiki.jenkins-ci.org/display/JENKINS/Thanks+for+using+OSX+Installer" target="_blank">official Jenkins documentation</a>