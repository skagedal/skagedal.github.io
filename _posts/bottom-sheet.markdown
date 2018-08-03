---
layout: post
title:  "Creating a Bottom Sheet"
date:   2018-07-31 00:00:00 +0200
---

The _Bottom Drawer_ or _Bottom Sheet_ is a user interface pattern that is used more and more on iOS, [showing up](https://twitter.com/lukew/status/1016388934933217282) in Apple’s own apps like Apple Music and Maps. But as of yet, there is no standard component for app developers to use. In this post I will show an approach to creating this kind of UI.  Instead of lengtby code snippets, I will insert links to appropriate commits of an [example project](https://github.com/skagedal/BottomSheet), if you want to follow along.  

We will create a simple Maps application, naturally using an `MKMapView`. On top of this, we want a table view that provides some shortcuts to various countries. We put these two main functionalities in a view controller each, [`MapViewController`](https://github.com/skagedal/BottomSheet/blob/4085bafa86cf3174fe2e3bddb2e8428d25d0e3bd/BottomSheet/MapViewController.swift) and [`CountriesTableViewController`](https://github.com/skagedal/BottomSheet/blob/4085bafa86cf3174fe2e3bddb2e8428d25d0e3bd/BottomSheet/CountriesTableViewController.swift). 

I like to use view controller composition to avoid ending up with everything in one big view controller, so we will build a `BottomSheetContainerViewController` that on construction takes two UIViewControllers, one that acts as the main, background UI (in our case, the map), and one that acts as the sheet. In our [very first implementation](https://github.com/skagedal/BottomSheet/blob/4085bafa86cf3174fe2e3bddb2e8428d25d0e3bd/BottomSheet/BottomSheetContainerViewController.swift), we just embed the sheet view controller on top of the main view controller with a fixed height. 

<img src="/images/bottom-sheet/initial.png" width="200" />

But now we want to be able to drag it up to cover the whole screen. While the table view in Apple Maps has three locations – fully covering the map, half-way covering the map or only showing a search box – we will only have two modes; fully covering and half-way covering the map. 

One way of doing this would be to install a pan gesture recognizer that changes the height constraint of the table view when panning. This approach has some problems, if we look at the Apple Maps as a goal for how we want things to behave. When overscrolling downwards, we want the same rubber banding behavior as we see in a scroll view. When we scroll the table view up to cover the map, given enough velocity, we want the table view to continue scrolling even after it has reached the top. Trying to make your own gesture recognizers play well with the table view’s can prove quite tricky. 

Instead, we’re going to use a different approach. All the gesture recognition and panning movements will be handled by the scroll view itself. To accomplish this, we will do the following:

* We make the table view cover the map view all the way up to the status bar, which we want to be the top position for the bottom sheet.  
* We give the table view a top content inset of 400. (In the end, this should not be a fixed constant, but we let it be for now.)
* We set the table view’s background color to `.clear`.
* We turn off the vertical scrolling indicator.

[Trying this out](https://github.com/skagedal/BottomSheet/commit/229c5955b4b7364208a3313539955cd392cbfe5d), we can already see the shape of how this is going to work. We have a bottom part that we can pull up and pull down, and it behaves mostly as we expect with regards to scrolling. But there are some problems. 

* The “background” of the drawer is now comprised of the table view cell’s backgrounds. When we scroll the cells all the way up, we see a hole in the background at the bottom. And the top doesn’t have the nice rounded corners look we want.
* We can now scroll the drawer to any position, stopping somewhere in between the “open” and “closed” position. Instead we want it to snap.
* We can’t interact with the map any more. 

We will now look at solutions to these issues in turn. 

## Background view

To make a background for the drawer that looks and behaves the way we want, we first make our table view cells transparent as well. We create a new UIView subclass called BottomSheetBackgroundView which implements the look for this background – as a [first version](https://github.com/skagedal/BottomSheet/commit/1d7037de627bb58452d06d9ca04e64aabffcaa98), we will just make it a solid white view. 

We need a method for the table view controller to communicate when it is scrolling so that we can update the position of this background view.  We will do this by a set of protocols.  

```swift
protocol BottomSheetDelegate: AnyObject {
    func bottomSheet(_ bottomSheet: BottomSheet, didScrollTo contentOffset: CGPoint)
}

protocol BottomSheet: AnyObject {
    var bottomSheetDelegate: BottomSheetDelegate? { get set }
}

typealias BottomSheetViewController = UIViewController & BottomSheet
```

Recall that the BottomSheetContainerViewController takes a parameter called `sheetViewController` of type UIViewController.  This will now instead be required to be a BottomSheetViewController – that is, a UIVIewController that also conforms to the protocol BottomSheet.  The BottomSheetContainerViewController will then set itself as the `bottomSheetDelegate` of the BottomSheetViewController, which is then expected to call the `bottomSheet(_:didScrollTo:)` method whenever the content offset is moved. 

But should we do that? Your first thought, when looking at the UIScrollView API:s, may be to implement the UIScrollViewDelegate's `scrollViewDidScroll` method. There's a problem with that however.  It only fires when the user scrolls, not when the change in content offset is initiated programmatically.  This is by design and a common pattern in UIKit – this way you can know why the delegate was called, and after all, you're already in programmatic control when you change properties yourself.  However, personally, I've found that for purposes of tracking content offset, it becomes difficult.  Suddenly things are out of sync. 

There is a different approach.  The `viewDidLayoutSubviews` method on the UITableViewController will _always_ be called whenever the scrolling changes. This is what we use in CountriesTableViewController. 

When the delegate method `bottomSheet(:didScrollTo:)` is called in the container view controller, we update the constant of a constraint that places the background sheet. This causes the desired effect.

Now we can [add a little style](https://github.com/skagedal/BottomSheet/commit/bb6a4f8ecfef6cb4c995dcdbbde59caf806a54e0) to our background view, and it's starting to look nice!


## Directing the taps

Next, we want to make it possible to interact with the map. This is easily done by implementing `hitTest(_:with:)` on our BottomSheetContainerView – and now we are happy that we chose to implement that as a subclassed UIView instead of putting everything in the view controller, as we now already have access to the things we need. 

We use the fact that the `sheetBackground` view corresponds to the are where we want the sheet view to be active.

```swift
    override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
        if sheetBackground.bounds.contains(sheetBackground.convert(point, from: self)) {
            return sheetView.hitTest(sheetView.convert(point, from: self), with: event)
        }
        return mainView.hitTest(mainView.convert(point, from: self), with: event)
    }
```

## Snapping

The way to make a scroll view (such as a table view) snap is to implement the `scrollViewWillEndDragging(_:withVelocity:targetContentOffset:)` delegate method. The `targetContentOffset` is both an input and an output to this method. It tells you where the content offset will end up after scroll deceleration has stopped, if you don't do anything.  But if you change it, that's where it will end up.  So that's what we will do when it is expected to fall in the disallowed region. 

```swift
    override func scrollViewWillEndDragging(_ scrollView: UIScrollView, withVelocity velocity: CGPoint, targetContentOffset: UnsafeMutablePointer<CGPoint>) {
        let targetOffset = targetContentOffset.pointee.y
        let pulledUpOffset: CGFloat = 0
        let pulledDownOffset: CGFloat = -maxVisibleContentHeight
        
        if (pulledDownOffset...pulledUpOffset).contains(targetOffset) {
            if velocity.y < 0 {
                targetContentOffset.pointee.y = pulledDownOffset
            } else {
                targetContentOffset.pointee.y = pulledUpOffset
            }
        }
    }
```

We're also setting the table view's `decelerationRate` to `.fast`. 

One problem you might notice is that if you don't have as many rows in the table view – try changing the constant `numberOfCountries` in CountriesTableViewController to 10 – it can no longer scroll all the way to the top, but stops when all cells are visible, and thus snap at the wrong place. We can fix this by making sure the `contentSize` of the table view is always at least a certain size. 

We add this into `viewDidLayoutSubviews`:

```swift
        if tableView.contentSize.height < tableView.bounds.height {
            tableView.contentSize.height = tableView.bounds.height
        }
```

## Conclusion

I think that the technique I have described here is a pretty nice way to get the desired behavior, although there are a few hoops you have to jump through.  Architecturally, there are some things that we might want to improve.  It would be nice could contain everything about the bottom sheet in the BottomSheetContainerViewController – currently we need to do some stuff in the table view controller itself.  While we could refactor some things, it seems difficult to make the table view controller completely ignorant of the fact that it's used as a bottom sheet.  

