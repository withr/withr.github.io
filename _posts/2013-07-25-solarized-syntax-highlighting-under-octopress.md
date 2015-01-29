---
layout: post
title: "Solarized Syntax highlighting under Octopress"
date: 2013-07-25 15:01
comments: true
categories: 
---

I started to use [Octopress](http://octopress.org/) as my blog framework from last year, until recently, I realized that the *Syntax higlighting* feature doesn't work. After spending half day to google the problem, now finally, it works. 



When embed a code [block](http://octopress.org/docs/plugins/codeblock/), *Octopress* allows we set which language the code belong to, like <code>lang:ruby</code> However, when I set the language, the *Octopress* can't generate any web page, just blank page. The problem is the *Syntax highlighting* feature need [pygments](http://pygments.org/), which run under *Python*, and I didn't install Python on my computer. Here is the solution:

+ Download [ActivePython](http://www.activestate.com/activepython/downloads) and install it. *ActivePython* is a commercial-grade distribution of Python, however you can use its Community Edition for free. The reason of choosing *ActivePython* is it contains *easy_install* which is not so easy to install on a 64-bit Windows, and it add Python's directory to environmental variable *PATH* by default. 
+ Once installed *ActivePython*, we can install *pygments* under Window Command Console or under Git console:

``` ruby
easy_install pygments
```

Now, re-run the *Octopress* (<code> rake generate; rake preview </code>), the pages should look OK. 

*Octopress* uses [Solarized](http://ethanschoonover.com/solarized) as its colors palette. We can custom the colors by modifing file <code> 'sass/custom/_colors.scss'</code>. For example, uncomment the following line to change the color sheme.

``` ruby
//$solarized: light;
``` 
