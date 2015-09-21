---
layout: post
title: "Hosting screenshots on GitHub"
description: ""
category:
tags: [github]
---

### The problem
I have some screenshots of an application that I want to put in my github repository's `README`. Fortunately, github provides a feature that can be used to solve this problem, called [github pages](http://pages.github.com/).

Github's pages service provides a lot of useful resources for hosting a website about your project. However, if you're just looking to host static content (such as images), pages will do the trick. Below, I'll show how to host content on github pages in a couple of minutes.

### Steps

<ol>
    <li>Create a new, empty branch called <code>gh-pages</code>
{% highlight bash %}
cd repo-name
git checkout --orphan gh-pages
git rm -rf .
rm -r .
{% endhighlight %}
    </li>
    <li>Add your static files<br/>
        Create an <code>images/</code> folder in the root of the branch, and put your screenshots in that folder.
    </li>
    <li>Commit and push <code>gh-pages</code> branch to github
{% highlight bash %}
git add images/*
git commit -m "Added static images/ folder"
git push -u origin gh-pages
{% endhighlight %}
    </li>
    <li>Switch back to your <code>master</code> repository
{% highlight bash %}
git checkout master
{% endhighlight %}
    </li>
    <li>Link to your images in your markdown file
{% highlight bash %}
# In your README.md file in your master branch:
![My Screenshot](http://username.github.com/repo-name/images/screenshot.png)
{% endhighlight %}
    </li>
</ol>

### That's it!

If you need to add new pictures to your repository, just switch to your `gh-pages` branch and push any other resources you need, then link to them in your README.
{% highlight bash %}
git checkout gh-pages
# Add content...
git commit -am "Added new images"
git push
git checkout master
{% endhighlight %}
