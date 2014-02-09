---
layout: post
title:  "Two simple tips to improve your command-line git experience"
date:   2013-12-21 20:13:53
description: "I wanted to put together a post with a couple of simple modifications you can make to your .gitconfig file that will improve readability when viewing changes to files in your Git working directory."
---

I wanted to put together a post with a couple of simple modifications you can make to your .gitconfig file that will improve readability when viewing changes to files in your Git working directory. By default, when you are in a Git working directory and you type
{% highlight javascript %}
git diff
{% endhighlight %}
in the command-line, you will see something like this:

![No Color](/img/2013-12-21-nocolor.png)

where changes are denoted on the left with or minus representing a line that has been removed and a plus representing a line that has been added. It seems that I replaced a blank line with an open div tag and later replaced another empty line with a close div tag. Though super-helpful, the way it is displayed takes some time to get used to and doesn't allow for changes to jump out at you right off.

# Pop of color

A simple fix is to enable colors on the command-line by modifying your .gitconfig file (which should be located in your home directory).  To open it, simply type this in the terminal:
<br/>
<br/>
<pre><code>open ~/.gitconfig</code></pre>
<br/>
Once open, insert these two lines in the file:
<br/>
<br/>
<pre><code>[color]
  ui = true
</code></pre>
<br/>
After that, the next time we type <code>git diff</code> in a Git working directory, we can see changes much more readily with a colorized UI.

![Color](/img/2013-12-21-color.png)

Now, deleted lines are shown in red and appended lines are in green, which to me makes it a lot easier to see what changes occured at first glance. <code>git status</code> output will be colorized as well.

# Opendiff and FileMerge

Another really helpful way to visualize changes made to you Git working directory (if you're on Mac OSX) is to use opendiff and FileMerge. Both tools should already be on your system if you've installed Xcode. To set these tools as your default 'difftool' viewer, go back into your .gitconfig file and append these lines:
<br/>
<br/>
<pre><code>[diff]
 tool = opendiff
[difftool]
 prompt = false</code></pre>
<br/>
After that simple change, if you type
<br/>
<br/>
<pre><code>git difftool</code></pre>
<br/>
in the terminal, it should run opendiff which will open the FileMerge GUI. In FileMerge, the old version of each file is shown in the top left (pre-change) and the current version is in the top right (post-change).  The final version is shown in the bottom panel with new changes highlighted in blue.

![OSX FileMerge](/img/2013-12-21-opendiff_filemerge.png)

Above, one can quickly observe that I've replaced a blank line with several lines including an open div tag and several lines of test text.  Once you're done looking at the changes, simply quit FileMerge to be taken back to the command-line prompt.

