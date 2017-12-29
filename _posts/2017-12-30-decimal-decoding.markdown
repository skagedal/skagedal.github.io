---
layout: post
title:  "JSON and Decimal numbers"
date:   2017-12-30 23:00:00 +0200
---

A JSON document can contain numbers, written in decimal notation. The standard does not specify any type for these numbers – a number is just defined syntactically as a sequence of digits, optionally followed by a dot and some more digits.  (There may also be a minus sign in the front and there may be a an exponent in the end – please see [the standard](http://json.org/) for the exact specification.)

A JSON parser should do the best job it can to preserve this information in the types available to the programmer.  In this blog post I am taking a look at the tools available on the Apple platforms, and explore a problem that occurs when using the `Decimal` type. 

## JSONSerialization

The `JSONSerialization` class, known as `NSJSONSerialization` in Objective-C, was the standard method of deserializing JSON up until the Swift 4 release.  How does it handle numbers?  Let's take a look at a few examples.  You may follow along in a playground. 


```swift
func parseJsonArray(_ jsonString: String) -> [Any] {
    let data = jsonString.data(using: .utf8)!
    return try! JSONSerialization.jsonObject(with: data, options: []) as! [Any]
}
func printInfo(_ items: [Any]) {
    for item in items {
        print("\(type(of: item)) - \(item)")
    }
}
printInfo(parseJsonArray("[5, 3.133]")
// Prints:
// __NSCFNumber - 5
// __NSCFNumber - 3.133
```

This being a typical Objective-C API, we're not specifying what types we expect; the framework parses into types it sees fitting, and it this case we get a private subclass of `NSNumber` called `__NSCFNumber`.  Exactly how that stores the number, we are not told. Do we always get this type for numbers? No:

```swift
printInfo(parseJsonArray("[1.23456789123456789]"))
// Prints:
// NSDecimalNumber - 1.23456789123456789
```

It has now switched to an `NSDecimalNumber`.  In all cases, whether we get an `__NSCFNumber` or an `NSDecimalNumber`, we can get a string representation of the parsed number that is exactly as how we wrote it in the JSON – no precision was lost. It presumably chooses an NSDecimalNumber where it can no longer fit tshe decimals in the mantissa of a Double.  

## JSONDecoder

With Swift 4, the `Codable` API arrived.  This gives us a new, more Swifty way of decoding JSON.  Now the programmer specifie the types to expect.  For most use cases, if you expect non-integer numbers, you will use the `Double` type.

```swift
func parseJsonDoubleArray(_ jsonString: String) -> [Double] {
    let data = jsonString.data(using: .utf8)!
    return try! JSONDecoder().decode([Double].self, from: data)
}
parseJsonDoubleArray("[5, 3.133]") // 5, 3.133
parseJsonDoubleArray("[1.23456789123456789]") // 1.2345678912345678
```

Note the truncation in the latter example – there are more digits than fit in the mantissa of a `Double`.  If you would need that many decimals, you might then want to choose the `Decimal` type instead. 

There are also other, probably more typical, situations where you may prefer `Decimal`. Decimal math is generelly preferred for example when dealing with currency, and you may want to decode directly into such a type. 

But alas:

```swift
func parseJsonDecimalArray(_ jsonString: String) -> [Decimal] {
    let data = jsonString.data(using: .utf8)!
    return try! JSONDecoder().decode([Decimal].self, from: data)
}
let decimals = parseJsonDecimalArray("[5, 3.133]")
decimals[1] // 3.132999999999999488
```

Hmmm.

## What's going on here? 

Before we get to this problem that appears to happen when we use _decimal_ floating point math, we have to talk a bit about _binary_ floating point. So, binary floating point – that's the kind we generally use, with types that programming languages typically call _float_ and _double_. These can be kind of difficult to really wrap your head around.  

We can write things like `let foo: Double = 0.1` all day long, and the compiler will be happy, so it's easy to forget that we can't actually represent that value as a binary floating point number. But just think about how the number 1/3 can't be represented in decimal notation – no finite sequence of 0.33333... will ever reach the value – and you can imagine how not all decimal quotients are representable in binary.  

Now, the whole point of a type like `Decimal` is to overcome this problem, when it is a problem. With a `Decimal`, you can precisely represent both 0.1 and 3.133.

```swift
let myDecimal = Decimal(string: "3.133")! // 3.133
```

There is no reason why a JSON decoder shouldn't be able to do this.  Unfortunately, `JSONDecoder` doesn't do JSON decoding of its own – it wraps `JSONSerialization`. One way to conclude that this is the case is to give it some invalid JSON as input, and you may recognize the errors as JSONSerialization errors, but we can also take a look at the source code – [here](https://github.com/apple/swift/blob/128092a7d60b57b8e6d69c8bda48f413b3d418b1/stdlib/public/SDK/Foundation/JSONEncoder.swift#L1060) is the call to JSONSerialization.

So, we are then not simply reading a decimal number string into a decimal number type, but converting via a binary floating point number.  In [the method](https://github.com/apple/swift/blob/d726bd85a24812065cf6164514144f9bbbf9fc5d/stdlib/public/SDK/Foundation/JSONEncoder.swift#L2309-L2319) with signature `func unbox(_ value: Any, as type: Decimal.Type) throws -> Decimal?` we can find how this conversion happens. 

```swift
        // Attempt to bridge from NSDecimalNumber.
        if let decimal = value as? Decimal {
            return decimal
        } else {
            let doubleValue = try self.unbox(value, as: Double.self)!
            return Decimal(doubleValue)
        }
```

The question is, why is this initializer doing such a poor job of converting the `Double` to the `Decimal`?  This is no longer a JSON issue – we can easily reproduce this in a playground:

```swift
let myDecimal2 = Decimal(Double(3.133)) // 3.132999999999999488
```
Or even just:

```swift
let myDecimal3: Decimal = 3.133 // 3.132999999999999488
```
That last one may be particularly surprising – the thing to remember here is that there are no Decimal literals; the `Decimal` types just conforms to the `ExpressibleByFloatLiteral`.  I don't think this is a great design decision; anyone who uses the `Decimal` type will do that for the particular reason of staying within decimal floating point logic; that person does not want a convenient way to initialize a Decimal that passes through a binary floating point conversion. 

Especially not when that conversion doesn't seem to be very good! It seems to me that, for any string representation of a decimal floating point number I can come up with (that does not overflow), this two-step conversion:

```swift
func decimalGood(from double: Double) -> Decimal {
    return Decimal(string: "\(double)")!
}
```

Gives better results than:

```swift
func decimalBad(from double: Double) -> Decimal {
    return Decimal(double)
}
```

I might very well be missing something here. This isn't new behavior, `[[NSDecimalNumber alloc] initWithDouble:3.133]` works the same way. 

