---
layout: post
title:  "Recreating UIButton"
date:   2017-04-11 16:15:28 +0200
---

In this post, we will recreate `UIButton`.  Not all of it, we're happy with just a clickable label.  

So, how is a UIButton created? Looking at the class hierarchy in the documentation, we see that `UIButton` directly inherits from `UIControl`.  

What does it consist of?  We can inspect its subviews and see that it has a label.  But not immediately, it is lazily created when accessed or on first layout. 

There is a small margin above and below the label.  Analyzing a co

* Placeholder for Table of Contents
{:toc}

## Simple 


{% highlight swift %}
{% endhighlight %}

