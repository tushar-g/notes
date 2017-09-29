# A Funny Thing Happened On The Way To This Array
**- Erica Sadun** 

Reference: [link](https://academy.realm.io/posts/try-swift-nyc-2017-erica-sadun-swift-flexibility-arrays/)


### Mutliple Views

Suppose we want a bunch of UIviews. The simplest and most naive way would be to write:

```swift
[UIView(), UIView(), UIView(), UIView(), UIView()]
```

However, there are a few problems with this way of declaring:
1. Fixed
2. Not reusable or generalizable
3. Hard to Audit - One has to count the number of items present in this array
4. Fails with more complex constructors.

**There are more swifty ways to generate this array.**

### Swifty Ways

##### A Procedural approach
```swift
let aBunch = 5 // Bunchiness shall vary

var views: [UIView] = []
for _ in 1 ... aBunch {
    views.append(UIView())
}
```
* More of a procedural style.
* Feels migrated, not native. 

##### A Functional approach
```swift
let views = (1 ... aBunch)
    .map { _ in UIView() }
```

* Functional approach
* But map has a specific meaning in math/fp. Map creates a relation between a value in domain to a value in range. 
* This shortcut kind of breaks the contract. It produces a different view for each member of the source range.

##### A Protocol approach
```swift
/// A type that can be instantiated without arguments
public protocol Constructible {
    init()
}

// Make views constructible
extension UIView: Constructible {}

extension Int {
    /// Return array containing `self` instances of a constructed type
    ///    extension NSObject: Constructible {}
    ///    3.of(NSObject.self)
    public func of<T: Constructible>(_ this: T.Type) -> [T] {
        return (0 ..< self)
            .map { _ in this.init() }
    }
}
```
* UIView() is a constructor. specifically `UIView.init()`
* We use swifty ways to create an operator `of` to construct types.
* By calling `of` on an Int we create an array of instances of that constructible type. `5.of(UIView.self)`
* There is an even cuter way of doing this. See below:

```swift
extension Int {
    /// Returns n constructed instances of this type
    public static func * <T: Constructible>(
        count: Int, this: T.Type) -> [T] {
        return count.of(this)
    }
}
```
* Now we can use `5 * UIView.self`

### Random Number Sequence

```swift
let randomNumber = Int(arc4random_uniform(UInt32(maxNumber)))
    
let randomNumberGenerator = { randomNumber }

randomNumberGenerator()
```
* Above is a random number generator
* Generators go along with sequences in swift.

```swift
let randomSequence = AnyIterator(randomNumberGenerator)

Array(randomSequence.prefix(5))
```
* This implementation gives us 5 random numbers. 
* An `Array` is constructed using an Interator and a generator.
* First few items are picked using prefix.
* prefix is method inside `AnyIterator` which spits out `AnySequence` up to the specified maximum length, containing the initial elements of the sequence.

```swift
Array(AnyIterator(UILabel.init).prefix(5))
```
* Similarly one can generate 5 UILabel views using `UILabel.init`

##### Using Initializers
* Array has a lot of built in Initializers. e.g:

```swift
/// Creates a new array containing the specified number of a single, repeated value.
///
/// - Parameters:
///   - repeatedValue: The element to repeat.
/// - count: The number of times to repeat the value passed in the `repeating` parameter. `count` must be zero or greater.

public init (repeating repeatedValue: Array.Element, count: Int)
```

* This initializer doesn't solve the randomNumber Generator problem. 
* This repeating initializer repeats exactly the same value for every single array position.
* Instead of five views, it spits out five of the exact same view. Instead of five random numbers, it constructs an array of five exact same number.
* We can make our own

```swift 
public extension Array {
    /// Creates a new array containing the specified number of values created by repeating a generating closure.
    ///
    /// - Parameters:
    ///   - count: The number of times to apply the closure. `count` must be zero or greater.
    ///   - generator: The closure to execute
    public init(count: Int,
                generator: @escaping () -> Element)
    {
        precondition(count >= 0, "Cannot initialize array with negative count")
        self.init(AnyIterator(generator).prefix(count))
    }
}
```
* This is an initializer. It’s part of a public extension, and it takes two parameters: a count and a generator
* Because the generator is the last argument, it can be used directly or as a trailing closure.
* Examples:

```swift
let views = Array(count: 5, generator: UIView.init)
// or
let views = Array(count: 5) { UIView() }

let numbers = Array(count: 5, generator: randomNumberGenerator)
// or
let numbers = Array(count: 5) { randomNumber }
```
