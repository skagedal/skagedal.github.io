---
layout: post
---

I'm having trouble with Xcode freezes.  What typically happens is that, all of a sudden, I can't run the app.  Xcode says "Index | Prebuilding..." and will does nothing when I press run.  It is otherwise responsive.  If I try to quit Xcode in this state, it will hang completely with the Spinning Beach Ball of Death.  I will have to force quit Xcode. 

The project where this happens is a mixed Objective-C and Swift projects.  I'm using Xcode 9.1-beta, but this also happened on 8.3.3 and Xcode 9.0.  Maybe it started happening on Xcode 8.3.3 after I installed the Xcode 9 betas, I'm not sure. 

I've been trying to see if I could gather more information by starting Xcode in debug mode.  In [this blog post](https://miqu.me/blog/2017/06/16/fixing-autocompletion-on-mixed-objective-c-and-swift-projects/), a method is mentioned where Xcode is started with a flag:

    /Applications/Xcode.app/Contents/MacOS/Xcode -ShowDVTDebugMenu YES

Then, there's supposed to be a submenu at Xcode -> Internal, but this does not show up for me. 

At least, starting Xcode from the terminal might give me some interesting information next time the freeze happens. 
