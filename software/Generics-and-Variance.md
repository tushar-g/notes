# Generics & Variance


Generic classes and interfaces are collectively known as _generic_ types. 
 
Each generic type defines a raw type, which is the name of the generic type used without any accompanying actual type parameters. E.g. in Generic Type List<E> , Raw type is List.  

My stamp collection. Contains only Stamp instances:

```java
// Not a good Idea. Compiles without error, Run time error when retrieving,
// if wrongly inserted.
private final Collection stamps = ... ;


// Parameterized collection type - typesafe
private final Collection<Stamp> stamps = ... ;

// Compilation Error (only if Parameterized): add(Stamp) in 
// Collection<Stamp> cannot be applied to (Coin)stamps.add(new Coin())
               ^
```

It is easy to imagine someone putting a java.util.Date instance into a collection that is supposed to contain only java.sql.Date instances. So using parametrized types, this is detected at compile Time itself.

**If you use raw types, you lose all the safety and expressiveness benefits of generics**

*Given that you shouldn’t use raw types, why did the language designers allow them?* **To provide compatibility**.


| List          |  List\<Object\>    | 
| ------------- | ------------------ |
| Opted out of Generic Type Checking         |  explicitly told the compiler that it is capable of holding objects of any type |
| `List list = new List<String>` is possible | `List<Object> list = new List<String>` is <mark>NOT</mark> possible                          |

* There are subtyping rules for generics.
* List<String> is a subtype of List, but not of List<Object>.
* As a result, we lose type safety when we use raw types.  


Sometimes it makes sense to use raw types when type information is not needed. for e.g:  

```java
// Use of raw type for unknown element type - don't do this!
static int numElementsInCommon(Set s1, Set s2) {       int result = 0;       for (Object o1 : s1)           if (s2.contains(o1))               result++;       return result;}
```

Here usage of raw types is dangerous. Instead we should use unbounded wild card type:

```java
// Unbounded wildcard type - typesafe and flexible
static int numElementsInCommon(Set<?> s1, Set<?> s2) {       int result = 0;       for (Object o1 : s1)           if (s2.contains(o1))               result++;       return result;}
```

**The wildcard type is safe and the raw type isn’t.**  
Reason: We can put any element in Raw type but we **can not put any element (other than null) into a Collection<?>**

Two exceptions where Raw types are used:  
1. **class literals**: List.class, String[].class, and int.class are all legal, but List<String>.class and List<?>.class are not.  
2. **instanceof operator**


