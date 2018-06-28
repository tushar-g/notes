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
* `List<String>` is a subtype of `List`, but not of `List<Object>`.
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

Shorthand: `X -> Y` means X is a subtype of Y

Let's consider following class heirarchy:

```java
public class Animal {
    String name;
    int troubleMakerRating;
    
    Animal(String name) {
        this.name = name;
        this.troubleMakerRating = new Random().nextInt();
    }
}

public class Dog extends Animal {
    Dog(String name) {
        super(name);
    }
}
    
public static class Cat extends Animal {
    Cat(String name) {
        super(name);
    }
}

```

According to Substitution Principle, If we have a collection of Animals, it's permissible to add a Dog or a Cat instance to it:  

```java
List<Animal> animals = new ArrayList<Animal>();
animals.add(new Animal("Lion"));
animals.add(new Dog("German Shepherd"));
animals.add(new Cat("RagDoll"));
```
Subtyping is in two forms in above code:  

1. assignment operation is permitted because `ArrayList<Animal> -> List<Animal>`
2. add operation is permitted because `Dog -> Animal`


Substitution Principle can also get us into trouble. The assignment (below) seems like a reasonable expectation, since `Dog -> Animal`, it must be that `List<Dog> -> List<Animal>`. WRONG!! 

```java
List<Animal> animals = new ArrayList<Dog>(); // Compilation ERROR !!

// if above assignment was permitted,
// below statement would also work, i.e adding a Cat instance to a list of Dog,
// causing run time issues.
animals.add(new Cat("Birman"))
```

The above code does not compile, because Substitution Principle does not apply. `List<Dog> !-> List<Animal>`.

But we want the Substitution to work some times. For ex:  

```java
// Consider a function to print names of an Iterable collection (of Animals)
void printNames(Iterable<Animal> animals) {
    animals.forEach(System.out::println);
}

ArrayList<Dog> myPets = new ArrayList<>();
dogs.add(new Dog("German Shepherd"));
dogs.add(new Dog("Bulldog"));
dogs.add(new Dog("Pug"));

printNames(myPets); // Compilation ERROR !!


// & What if we need to compare two dogs using a generic Animal Comparator
static void compareDogs(Comparator<Dog> comparator) {
    Dog myPetPoodle = new Dog("Poodle");
    Dog myPetBeagule = new Dog("Beagle");
    if (comparator.compare(myPetPoodle, myPetBeagule) > 0) {
        System.out.println("My German Shepherd is a trouble maker");
    } else {
        System.out.println("My Pug is a trouble maker");
    }
}

Comparator<Animal> troubleMakerComparator = Comparator.comparingInt(a -> a.troubleMakerRating);
compareMyPetDogs(troubleMakerComparator); // Compilation ERROR !!

```
We use variance to solve this.

# Variance

Variance refers to how subtyping between more complex types relates to subtyping between their components. For example, how should a list of `Cats` relate to a list of `Animals`, Or how should a function returning `Cat` relate to a function returning `Animal`



There are four possible variances:  
1. **Covariant** : Preserves ordering of sybtyping relation. i.e. if `A -> B`, then `Box<A> -> Box<B>`  
2. **Contravariant** : Reverses the ordering of subtyping relation. i.e. if `A -> B`, then `Box<B> -> Box<A>`  
3. **Bivariant** : If both of the above applies. i.e. if `A -> B`, then `Box<A> -> Box<B>` & `Box<B> -> Box<A>`  
4. **Invariant / Non-variant** : Neither of above applies.

### Why do we need Covariance & ContraVariance

