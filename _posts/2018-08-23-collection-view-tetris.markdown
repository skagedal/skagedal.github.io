---
layout: post
title:  "Collection View Tetris"
date:   2018-08-23 23:01:28 +0200
---

I'd like to tell the story of how exploring table view animations led me to this beautiful use of Xcode's color literals.

<figure>
<img src="/images/collection-view-tetris/termino-color-literals.png" alt="Image of data structure in Swift that use color literals to specify Tetris shapes." width="400px" />
</figure>

A while back, I was investigating the animations in UITableView and UICollectionView.  As you may be aware, in a table view you may do the following six kinds of animations:

* Inserting rows
* Deleting rows
* Moving rows
* Inserting sections
* Deleting sections
* Moving sections

The same set of animations, with items instead of rows, are available for collection views. These animations can also be batched, so that several operations are animated at once. 

If you've ever used these methods, you may have become acquainted with the dreaded _NSInternalInconsistencyException_. The thing is, whenever you ask a table view to – for example – insert one row in a section, you have to make sure that when it calls your data source's `numberOfRows(inSection:)`, it has increased the value by one. 

When you start out with table views and iOS programming, it may happen that you try to add logic in your data source methods that may look a little something like this: 

```swift
func numberOfRows(inSection section: Int) -> Int {
    if section == animalsSection {
        return animals.count + self.isAddingAnimal ? 1 : 0 
    }
}
```

This kind of approach always ends in tears.  It very quickly gets very hard to keep all the state consistent.  Instead, you always want a model that represents the full state – an array, typically – and if you want to animate from one state to another, use a _differ_ to calculate what rows and sections to insert, delete or move.  

There are several good such tools around.  They typically use the _Longest Common Subsequence_ algorithm to perform the diff, which is what you should use for good performance characteristics.  (Please tweet me the best ones and I'll update this text.) 

What I became preoccupied with was the question of how to make such a diffing method that handled both items (rows) and sections and worked for _any_ input. Basically, I wanted a function like:

```swift
func diff(from oldSections: [(SectionType, [ItemType])], 
          to newSections: [(SectionType, [ItemType])]) -> BatchChanges
```
And then this `BatchChanges` would contain everything that you needed to send to the animation methods. It turned out that this was quite hard, because there were diffs that were impossible to perform in just one step – for example, when an item moves from one section to another, and that section also moves at the same time.  Instead, you have to:

* first diff the _sections_ from old to new,
* then _apply_ that diff to your old _items_ (let's call the result of that the patched items),
* then diff your items from your patched items to the new items. 

Then _first_ perform the section animations – and while that happens make sure to return the per-section item counts from the _patched_ items – _then_ perform the item animations.

Phew. 

## This is boring, let's play Tetris

So anyway, after I had my very general `DataSource<SectionType, ItemType>` that could handle animation of changes from any state to any other, I wanted to do something crazy with it, to kind of see what you can do. 

I got the idea of a Collection View Tetris. Here it is. 

<figure>
<img src="/images/collection-view-tetris/tetris.gif" alt="Screencap of me playing Tetris, badly." />
</figure>

Here, every Tetris row is a collection view section consisting of ten cells.  Each cell is a standard `UICollectionViewCell` with only the background color set. The animations are really just cells switching places.  And when you get _Tetris_, a section is deleted.  But this all happens because of the differ – the [Tetris game logic](https://github.com/skagedal/SKRBatchUpdates/blob/master/SKRBatchUpdatesDemo/SKRBatchUpdatesDemo/Tetris.swift) is written with no awareness of collection views.

I thought it was a fun hack.  It's been sitting around on my computer for a while now, waiting for me to do a presentation that never happened, so now I thought I'd just publish it.  I don't actually use the DataSource, because I haven't needed anything more than just animating rows within a single section.  Keep it simple, huh?

For all the fun, [see the source on Github](https://github.com/skagedal/SKRBatchUpdates).  The color literals?  Unfortunately, they do not look as awesome in Github as they do in Xcode, and I don't really recommend the use of them.  Nor do I recommend that you use a UICollectionView for your next game.  I do recommend having fun with things. 




