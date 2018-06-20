# Generics & Variance

Generic classes and interfaces are collectively known as _generic_ types. 
 
Each generic type defines a raw type, which is the name of the generic type used without any accompanying actual type parameters. E.g. in Generic Type List<E> , Raw type is List.  

My stamp collection. Contains only Stamp instances:

```java
// Not a good Idea. Compiles without error,
// Run time error when retrieving, if wrongly inserted.
private final Collection stampsUntyped = ... ;

// Compilation Error
stampsTyped.add(new Coin())
                    ^

// Parameterized collection type - typesafe
private final Collection<Stamp> stampsTyped = ... ;

// No compilation error, Runtime Error when accessed
stampsUntyped.add(new Coin())
```

It is easy to imagine someone putting a java.util.Date instance into a collection that is supposed to contain only java.sql.Date instances. Parametrized types would prevent this at compile time.

**If you use raw types, you lose all the safety and expressiveness benefits of generics**

*Given that you shouldn’t use raw types, why did the language designers allow them?* **For Migration compatibility**.


| List          |  List\<Object\>    | 
| ------------- | ------------------ |
| Opted out of Generic Type Checking         |  explicitly told the compiler that it is capable of holding objects of any type |
| `List list = new List<String>` is possible | `List<Object> list = new List<String>` is <mark>NOT</mark> possible                          |

* There are subtyping rules for generics.
* List\<String\> is a subtype of List, but not of List<Object>.
* As a result, we lose type safety when we use raw types.  


Sometimes it makes sense to use raw types when type information is not needed. for e.g:  

```java
// Use of raw type for unknown element type - don't do this!
static int numElementsInCommon(Set s1, Set s2) {       int result = 0;       for (Object o1 : s1)           if (s2.contains(o1))               result++;       return result;-}
```

Here usage of raw types is dangerous. Instead we should use unbounded wild card type:

```java
// Unbounded wildcard type - typesafe and flexible
static int numElementsInCommon(Set<?> s1, Set<?> s2) {       int result = 0;       for (Object o1 : s1)           if (s2.contains(o1))               result++;       return result;}
```

**What does this question mark (Unbounded Wild Card Type) buys us?**   
- The wildcard type is safe and the raw type isn’t.  
- Since Generics in Java are invariant, we can put anything inside a Set but Compiler prohibits us to put anything other than a null inside a Set<?>.


# Subtyping

Its a key feature of Object-oriented languages such as Java. If we `extend` or `implement` a certain Type, the more specific type (child) can fit in a bigger container (Parent). For example, in Java: 

* Integer is a subtype of Number
* Double is a subtype of Number
* String is a subtype of Object
* String[] is a subtype of Object[]
* ArrayList<E> is a subtype of List<E>

Subtype is also transitive. i.e. if A extends B, B extends C, then A extends C.
In Java, every reference type is a subtype of Object. Also, trivially, every type is a subtype of itself.

**Substitution Principle**:
Whereever a value of one type is expected, one may provide a value of any subtype of that type.


# Variance

Variance refers to how subtyping between more complex types relates to subtyping between their components. For example, how should a list of `Cats` relate to a list of `Animals`, Or how should a function returning `Cat` relate to a function returning `Animal`

Shorthand: `X -> Y` means X is a subtype of Y

There are four possible variances:  
1. **Covariant** : Preserves ordering of sybtyping relation. i.e. if `A -> B`, then `Box<A> -> Box<B>`  
2. **Contravariant** : Reverses the ordering of subtyping relation. i.e. if `A -> B`, then `Box<B> -> Box<A>`  
3. **Bivariant** : If both of the above applies. i.e. if `A -> B`, then `Box<A> -> Box<B>` & `Box<B> -> Box<A>`  
4. **Invariant / Non-variant** : Neither of above applies.

