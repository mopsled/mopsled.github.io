---
layout: post
title: "How To Build optool on Mac OS 10.11"
description: ""
category:
tags: [optool, xcode]
---

optool
---

`optool` is a tool for editing MachO executables load commands. This post outlines how to build optool from source.

Steps
---

### Clone from git

{% highlight bash %}
git clone https://github.com/alexzielenski/optool.git
{% endhighlight %}

Make initialize `optool`'s submodules:

{% highlight bash %}
cd optool/
git submodule update --init --recursive
{% endhighlight %}

If you're running into the errors:

> 'FSArgumentParser/ArgumentParser/FSArguments.h' file not found

or

> No such file or directory: 'optool/optool/FSArgumentParser/CoreParse/CoreParse/Parsers/CPShiftReduceParsers/CPItem.m'

run the `git` command above to recursively download `optool`'s submodule requirements.

### Change `optool`'s deployment target

Unless you're running Mac OS X 10.9, `optool`'s default project settings cause these build errors:

> The run destination My Mac is not valid for Running the scheme 'optool'.

or

> Check dependencies
> error: There is no SDK with the name or path '/Users/mopsled/Source/optool/macosx10.9'

To fix it, the project's build settings need to be changed.

Open the `optool` project in XCode. Click on the `optool` project in the project organizer:

![](/assets/images/optool-build/optool-select.png)

Next, click on the `optool` target:

![](/assets/images/optool-build/optool-target.png)

And change it to the `optool` project:

![](/assets/images/optool-build/optool-project-select.png)

In the `optool` project, select **Build Settings**:

![](/assets/images/optool-build/select-build-settings.png)

Select the **Base SDK** setting and change it to **Latest OS X**

![](/assets/images/optool-build/select-latest.png)

Finally, click the Run button to compile `optool` for your Mac:

![](/assets/images/optool-build/run-build.png)

### Locate the `optool` binary

After the project builds, select the `optool` Product in the project organizer and click `Show in Finder`:

![](/assets/images/optool-build/show-in-finder.png)

Finder should open to the built copy of optool.

![](/assets/images/optool-build/optool-finder.png)</li>

You're done!
