---
layout: post
title:  "Creating Designable Views"
date:   2016-06-18 21:53:28 +0200
---

Should I do it in code, or should I do it in Interface Builder?  That's a question I struggled a lot with as a beginner iOS developer, and I think the same goes for many of us.  

* Placeholder for Table of Contents
{:toc}

## The Emoticon View Example

![How did this run feel? Bad, Okay, Great.](/images/creating-designable-views/how-did-this-run-feel.jpg)

The Runkeeper app has a view that is displayed after a run is completed, that asks you how you feel about the run. You answer by tapping emoticons representing Bad (frowning face), Okay (neutral face) or Great! (smiley face). How would you implement this? The simplest way is often the best, and in this case that means importing image assets into your project use them in your storyboard. But let's say you need more flexibility. You might want to add some animations that can't be done with the static image, or you want to configure the exact smileyness of the emoticon. Let's see how to do this with a custom view.

First, create a iOS project in Xcode using the _Single View Application_ template. We will use Swift as a language. Then, create a new class for our emoticon. Make it a subclass to `UIView` -- let's call it `FaceView`. In the first iteration of our class, we'll implement our face by overriding `drawRect(_:)`. This method has some drawbacks, which we'll discuss later, but is the simplest way to get going with designable views. Let's start off by just drawing the outline of the face.  This is how the Swift file will look:

{% highlight swift %}
import UIKit

class FaceView: UIView {
    override func drawRect(rect: CGRect) {
        UIColor.darkGrayColor().set()
        let bounds = self.bounds.insetBy(dx: 1.0, dy: 1.0)
        let headPath = UIBezierPath(ovalInRect: bounds)
        headPath.lineWidth = 2.0
        headPath.stroke()
    }
}
{% endhighlight %}

Now, drag a new view to the center of your storyboard's default view controller. (Yes, a plain UIView - I find the quickest way to find this in the object browser by searching for "uiview", by the way). Give it some constraints: fixed width and height of, say, 100 points, and center it vertically and horizontally. Then set its class to `FaceView`.

Run your app, and you should see a circle in the center of the screen. 

Set up your screen so that you have the the storyboard in the assistant view, and `FaceView.swift` in your main view.  Now get ready for the magic.  Just add the `@IBDesignable` keyword right before the class:

{% highlight swift %}
@IBDesignable
class FaceView: UIView {
{% endhighlight %}

Boom, the circle is now visible in your storyboard.  And now you can have a lot of fun coding, seeing and interacting with the results in real time.  Well, interacting -- we can resize and move the frame and see how our code reacts to that.  But there's another thing that goes hand-in-hand with IB-designables, and that's IB-inspectables.  Let's add one of those. 

{% highlight swift %}
@IBDesignable
class FaceView: UIView {
    @IBInspectable var lineWidth : CGFloat = 2.0;
    
    override func drawRect(rect: CGRect) {
        UIColor.darkGrayColor().set()
        let inset = self.lineWidth / 2.0;
        let bounds = self.bounds.insetBy(dx: inset, dy: inset)
        let headPath = UIBezierPath(ovalInRect: bounds)
        headPath.lineWidth = self.lineWidth
        headPath.stroke()
    }
}
{% endhighlight %}

Now you can go to the Attributes Inspector for the Face View and change the thickness of its outline. 

## Debugging Designable Views

## Adding Subviews

We would like to add some animation to our emoticon.  We would like the eyes to blink when you select one of the faces.  We want to do this by having each eye as a subview of the Emoticon view, instead of drawing them in `drawRect:`.  But where do we create these ?  

## Designing the Designable View

Using a nib as a designable view. 

## Common Helpers

Exposing the layer's corner radius, border with etc.  Setting background color to a pattern.  

You may do this as an extension or category on UIView, which makes these properties available on all UI
Doing it as a a category or extension.

## Using Designable Views as Style Classes?

## Put your Designables in an Embedded Framework

