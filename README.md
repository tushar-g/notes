# Notes
Notes & Summary of everything I shall read.

```swift
class MyClass { }
extension MyClass {
    
    // A Computed Property -> No backing Field
    var foobar: String { return "FooBar" }
    
    // On Type
    static func foo() {
        print("Foo")
    }
    
    // On Instance
    func bar() {
        print("Bar")
    }
}

MyClass.foo()
MyClass().bar()
MyClass().foobar

```