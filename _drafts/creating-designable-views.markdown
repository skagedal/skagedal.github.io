---
layout: post
title:  "Creating Designable Controls"
date:   2016-06-18 21:53:28 +0200
---
The Runkeeper app has a view that is displayed after a run is completed, that asks you how you feel about the run. You answer by tapping emoticons representing Bad (frowning face), Okay (neutral face) or Great! (smiley face). How would you implement this? The simplest way is often the best, and in this case that means importing image assets into your project use them in your storyboard. But let's say you need more flexibility. You might want to add some animations that can't be done with the static image, or you want to configure the exact smileyness of the emoticon. Let's see how to do this with a custom view.

First, create a iOS project in Xcode using the _Single View Application_ template. We will use Swift as a language. Then, create a new class for our emoticon. Make it a subclass to `UIView` -- let's call it `FaceView`. In the first iteration of our class, we'll implement our face by overriding `drawRect(_:)`. This method has some drawbacks, which we'll discuss later, but is the simplest way to get going with designable views. Let's start off by drawing the outline of the face:


{% highlight swift %}
override func drawRect(rect: CGRect) {
    UIColor.darkGrayColor().set()
    let bounds = self.bounds.insetBy(dx: 1.0, dy: 1.0)
    let headPath = UIBezierPath(ovalInRect: bounds)
    headPath.lineWidth = 2.0
    headPath.stroke()
}
{% endhighlight %}

