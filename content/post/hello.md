+++
title = "Hello World"
description = ""
tags = [
    "hugo",
    "programming",
    "meta",
    "web",
]
date = "2017-12-25"
categories = [
    "Programming",
    "Meta",
]
menu = "main"
+++

Take Network is now a real, live network where you will presumably be able to ingest takes. I can think of no better way to christen such a momentous occasion than with a recap of how I got this all online.
<!--more-->
## Structure

Take Network is a blog built on the [Hugo](http://gohugo.io) platform. I am not yet an expert on static site generators, but after playing around with WordPress it was very quickly apparent that a full database-backed service was probably overkill for a simple blog. Hugo seems to be the most popular project in this segment, with its main competitor being Jekyll.

I originally wanted to host this site locally, but since I already have a number of other services running on my Digital Ocean droplet it was going to be too messy to try to route everything nicely with Apache. After a bit of research on my available options, I found a great [guide](https://gohugo.io/hosting-and-deployment/hosting-on-github/) on the Hugo website on how to host via GitHub Pages and folks-- it works extremely well. Namecheap updated my `blog.` sudomain configuration almost instantly and all of the pathing resolved itself "automatically". Might want to file this integration under [Black Magic Fuckery](https://www.reddit.com/r/blackmagicfuckery/).

## Hugo Tricks

One thing that should unfortunately _not_ be filed under Black Magic Fuckery is Hugo's documentation. The configuration required is minimal, but it tends to be very unclear what the workflow for building static content is supposed to be. One issue particular that I've already found endlessly frustrating is that if you're storing your site on GitHub, the actual contents of your cloned theme repo will not commit, and thus will not be pulled in on a `git clone`. This gives you an empty directory with your theme name, which is enough to work with such that Hugo will _not_ complain about missing the theme but also completely fail to build your site. Wonderful!

Another issue I've had is that tying the pathing for CSS to your `config.toml`'s `baseurl` is incredibly confusing, especially since a person getting started will likely not be working right off of their final hosting destination. In my case, CSS worked fine when running `hugo server`, but not when opening `index.html` directly (which presumably will _never_ resolve CSS correctly). Who's going to push this stuff to a server in order to verify that their `baseurl` was configured correctly when the pages look totally broken locally?

I still need to figure out a good mechanism for updating the site content- right now, I'm hoping that as long as I remember to re-generate the static content into `docs/`, GitHub Pages hosting should be relatively painless. However, this is definitely cumbersome in a scenario where I just need to fix quick typo. If you've got any tips or tricks, feel free to get at me with the contact buttons on the top right.

## Writing Markdown

Lastly- a few words on writing in Markdown, since I'm familiar with the syntax but don't quite have a workflow set up yet.

Generally, (what I think should be) my approach for Markdown is to get a split-view editor where I can write on one side and see the resulting text on the other. This seems likely a fairly natural approach, since you'd want to keep the lines that you've literally typed separate from the formatted prose, right?

Apparently not everyone agrees. I found the [Typora](https://typora.io) editor and thought that it looked pretty sleek and distraction-free, but couldn't find any split functionality. [It turns out](https://github.com/typora/typora-issues/issues/70) that they're pretty vehemently against that idea, instead shooting for a WYSIWYG philosophy. But that only works when you've got font and size options like in a real document editor, right? Taking that approach in an editor where you're writing "code" is like writing HTML and automatically having it parse and render as you type. But that's just my opinion anyway.

Instead I've settled for the moment on Visual Studio Code. It seems to do the trick so far, but obviously it's a much more complicated tool. I do like that it came with the Solarized color schemes out of the box, and I might find the Git integration helpful. Interestingly,  it seems to fuck with my NVidia Experience overlays-- the [official greybeard response](https://github.com/Microsoft/vscode/issues/37104) seems to be dismissive of such Gamer Issues.

One last note on writing in Markdown- Hugo uses a meta block called [Front Matter](https://gohugo.io/content-management/front-matter/) like the below to store page metadata:

    +++
    title = "Hello World"
    description = ""
    tags = [ "hugo", "programming", "meta", "web", ]
    date = "2017-12-25"
    categories = [ "Programming", "Meta", ]
    menu = "main" 
    +++

Markdown seems to not understand this sort of construct- it's not totally freaking out or anything like that, but interpreting it as inline prose makes it kind of awkward to edit at a glance. Presumably there's some sort of plugin for VSCode to pretty-print Hugo's Front Matter, but I'm sure I'll figure that out in the weeks to come.
