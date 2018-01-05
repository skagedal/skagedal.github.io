---
layout: post
title:  "Specifying device in simcd"
date:   2018-01-05 17:00:00 +0200
---

In my [previous post](https://skagedal.github.io/2018/01/02/simcd.html), I created a `zsh` plugin for changing current directory to the data container of an app in the currently booted Xcode simulator.  However, as of Xcode 9, we can have several simulators open at once.  We therefore need to be able to choose the simulator that `simcd` should operate on.  

Rewriting the `simcd` function to take another option that is passed on to `xcrun simctl` for the device argument is easily done.  But of course, we want this to be completable as well.  This post presents a failed attempt – or a work in progress – to do so. 

## Listing booted devices

We can list the available (and unavailable) devices with `xcrun simctl list devices`.  If we tack on a `-j`, we get the result in JSON.  This JSON looks like this:

```json
{
  "devices" : {
    "iOS 9.1" : [
      {
        "state" : "Shutdown",
        "availability" : "(available)",
        "name" : "iPhone 6s 9.1",
        "udid" : "0BF0D4DD-62CB-4139-9E2C-BCDEF6C80E2C"
      },
      {
        "state" : "Shutdown",
        "availability" : "(available)",
        "name" : "iPhone 6s Plus",
        "udid" : "A41BC93D-687E-477F-90FA-F1F070370DFB"
      }
    ],
    "tvOS 11.2" : [

    ],
    "More OS versions": "continued..."
   }
 }
``` 

So we have a dictionary ("object" in JSON terms, but I find that terminology confusing) with an entry called `devices`, which is itself a dictionary where keys have OS versions and the values contain all your simulators. 

We want the devices where the property with the key `"state"` has the value `"Booted"`. (You may think that we might want to allow specifying any simulator, but it turns out we can only run the simctl command that gives us a data container directory for simulators that are running.)

Here's how we can use a Ruby expression to print these entries:

```shell
xcrun simctl list devices -j | \
	ruby -r json -e 'puts JSON.parse(STDIN.read)["devices"] \
		.values \
		.flatten \
		.select{|x| x["state"] == "Booted"}'
```

You will get the Ruby hash for each running simulator device, where  we can get the UDID and user-specified name for the device.

## Getting devices names

Now, time for a brief discussion on what the UX should be here, given the constraints we have.  How should we expect the user to specify a device?  The `xcrun simctl` commands that take a device will allow either a special name such as `"booted"` (that we've used), an UDID or the user-defined name.  We could also allow all these, but we should optimize for one case to make the completability super-nice.  

UDID has the nice property of being unique, but isn't very user friendly.  You probably know that you're running an iPhone 8 and an iPhone SE, that's what you want to specify.  So, name, then.  Let's just decide that the user has to make sure that each simulator device has an unique name.  That might be a good idea anyway.  You can rename them in Xcode's "Devices and Simulators" window; I now have a simulator called "iPhone 8 11.2" and one called "iPhone X 11.2".

 To our Ruby expression above, we add:

```ruby
		.map{|x| x["name"]}
```

Which gives me the following output:

```
iPhone 8 11.2
iPhone X 11.2
```

## Shell programming is really annoying

Shell programming is really annoying, and one of the main reasons for that is the way spaces are used to separate things.  If I give the above to a zsh completion definition, it will see six words: "iPhone", "8", "11.2", "iPhone", "X" and "11.2". That's not what we want. 

I tried modifying the Ruby so that it encloses each name in quotation marks. But zsh still wasn't happy. 

I am giving this a break for now and we'll see if I get the inspiration to continue. 

