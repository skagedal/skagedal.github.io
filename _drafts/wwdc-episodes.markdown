---
layout: post
title:  "WWDC Episodes"
---

Me taking notes on Apple videos. 

## Watched videos

### WWDC 2017

* [Auto Layout Techniques in Interface Builder](https://developer.apple.com/videos/play/wwdc2017/412/)
  <br/>
  Common techniques for changing layout constraints. Trackig touches. Dynamic Type - I learned how to use the Accessibility Inspector to try out various sizes, very cool. 

* [What's New in iTunes Connect](https://developer.apple.com/videos/play/wwdc2017/302/)
  <br />
  More interesting for project managers than for developers, perhaps. 
* [Customized Loading in WKWebView](https://developer.apple.com/videos/play/wwdc2017/220/)
  <br />
  This is great stuff that I think we're going to have good use of in a project at work.  We have a problem with WKWebView not immediately being aware of cookies that it should share with an NSURLSession. We've had to work around it by setting the cookies manually on the initial load request, and then _also_ by injecting some JavaScript that will set the cookies on following requests.  While this seems like a bug tog me, it seems that a simpler workaround should be possible using the new API:s to set cookies in a WKWebView.  The new content loading mechanism through custom URL schemes also sounds great.
* [Updating Your App for iOS 11](https://developer.apple.com/videos/play/wwdc2017/204/)
  <br />
  Lots of really useful stuff, for example about layout margins and safe area. We should consider whether to start using "preserve PDF data" in our apps.  Is that backwards compatible?  Also, the new swipe actions, love those. 
* [Introducing PDFKit on iOS](https://developer.apple.com/videos/play/wwdc2017/241/)
   <br />
   We'll be using this.
* [What's New in Cocoa Touch](https://developer.apple.com/videos/play/wwdc2017/201/)

### Fall 2017

* [Building Apps for iPhone X](https://developer.apple.com/videos/play/fall2017/201)
  <br />
  Useful code examples on how to use the navigation item's search controller.
* [Designing for iPhone X](https://developer.apple.com/videos/play/fall2017/801/)

## Currently watching

* [Modern User Interaction on iOS](https://developer.apple.com/videos/play/wwdc2017/219/)

## Videos to watch

### WWDC 2017

* [What's New in Core Data](https://developer.apple.com/videos/play/wwdc2017/210/)
* [Designing Sound](https://developer.apple.com/videos/play/wwdc2017/803/)
* [Writing Great Alerts](https://developer.apple.com/videos/play/wwdc2017/813/)
* [App Startup Time: Past, Present, and Future](https://developer.apple.com/videos/play/wwdc2017/413/)
* [Localizing with Xcode 9](https://developer.apple.com/videos/play/wwdc2017/401/)
* [Understanding Undefined Behavior](https://developer.apple.com/videos/play/wwdc2017/407/)
* [Modernizing Grand Central Dispatch Usage](https://developer.apple.com/videos/play/wwdc2017/706/)
* [What's New in Testing](https://developer.apple.com/videos/play/wwdc2017/409/)
* [High Efficiency Image File Format](https://developer.apple.com/videos/play/wwdc2017/513/)
* [Introducing MusicKit](https://developer.apple.com/videos/play/wwdc2017/502/)

