---
layout: post
title:  "Magical Logging in bash and zsh"
date:   2015-06-09 21:30:28
comments: true
---

<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML-full" charset="utf-8"></script>
<script type="text/javascript">
MathJax.Hub.Config({
  tex2jax: {
    inlineMath: [['$','$'], ['\(','\)']],
    processEscapes: true
  }
});
</script>

##Disclaimer
So far this snippet has only been tested on OSX. It should work on any \*nix with some fiddling though!

##References
* This [post from **Simon Coffey**](http://urbanautomaton.com/blog/2014/09/09/redirecting-bash-script-output-to-syslog/) gave me the idea to try and log **all** stderr from the shell to syslog or another logging facility, not just from an individual script

* This response on [ask ubuntu](http://askubuntu.com/questions/93566/how-to-log-all-bash-commands-by-all-users-on-a-server) shows how to capture all commands to syslog

##Motivation
I recently decided it'd be fun to log all of my and my coworker's commands and stderr from our zsh sessions. Not in an Orwellian way, but because they're all data scientists and it'd generate a lot of cool data! Also, I've been experimenting with the awesome [**elasticsearch, logstash, and kibana stack**](https://www.elastic.co/), and what better way to test it than to fill it up with a bunch of logs!

After some intense googling, there was nothing that did exactly what I wanted.

##zsh version
Add this to the bottom of your .zshrc file
{% highlight bash %}
precmd() { eval "$PROMPT_COMMAND" }
export PROMPT_COMMAND='RETRN_VAL=$?;logger "$(fc -l -1 | sed "s/^[ ]*[0-9]+[ ]*//" ) [$RETRN_VAL]"'
exec 2> >(tee >(logger -t $(echo $USER).stderr))
{% endhighlight %}

##bash version
Add this to the bottom of your .bashrc file
{% highlight bash %}
export PROMPT_COMMAND='RETRN_VAL=$?;logger "$(history 1 | sed "s/^[ ]*[0-9]+[ ]*//" ) [$RETRN_VAL]"'
exec 2> >(tee >(logger -t $(echo $USER).stderr))
{% endhighlight %}

##Explanation
I'll break down the zsh version since that's my primary shell. Zsh doesn't have the PROMPT_COMMAND magic like bash, so the line
{% highlight bash %}
precmd() { eval "$PROMPT_COMMAND" }
{% endhighlight %}
uses zsh's "precmd" hook to emulate the behavior. 



