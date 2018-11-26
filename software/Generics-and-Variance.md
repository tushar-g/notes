# Generics & Variance

Generic classes and interfaces are collectively known as _generic_ types. 
 
Each generic type defines a raw type, which is the name of the generic type used without any accompanying actual type parameters. E.g. in Generic Type List<E> , Raw type is List.  

My stamp collection. Contains only Stamp instances:

```java
// Not a good Idea. Compiles without error,
// Run time error when retrieving, if wrongly inserted.
private final Collection stampsUntyped = ... ;

// No compilation error, Runtime Error when accessed
stampsUntyped.add(new Coin())

// Parameterized collection type - typesafe
private final Collection<Stamp> stampsTyped = ... ;

// Compilation Error
stampsTyped.add(new Coin())                    
                    ^
```

It is easy to imagine someone putting a `java.util.Date` instance into a collection that is supposed to contain only `java.sql.Date` instances. Parametrized types would prevent this at compile time.

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
static int numElementsInCommon(Set s1, Set s2) {
   int result = 0;
   for (Object o1 : s1)
       if (s2.contains(o1))
           result++;
   return result;-
}
```

Here usage of raw types is dangerous. Instead we should use unbounded wild card type:

```java
// Unbounded wildcard type - typesafe and flexible
static int numElementsInCommon(Set<?> s1, Set<?> s2) {
   int result = 0;
   for (Object o1 : s1)
       if (s2.contains(o1))
           result++;
   return result;
}
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

But we want the Substitution to work some times. For example, Consider a function to print names of an Iterable collection (of Animals):


```java
void printNames(Collection<Animal> animals) {
    animals.forEach(System.out::println);
}

List<Dog> myPets = new ArrayList<>();
dogs.add(new Dog("German Shepherd"));
dogs.add(new Dog("Bulldog"));
dogs.add(new Dog("Pug"));

printNames(myPets); // Compilation ERROR !! Incompatible Types

```
This does not compile. Even though we are not writing anything to the `Iterable<Animal>`, we are only accessing (reading) from the collection.

& What if we need to compare two dogs using a generic Animal Comparator

```java
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
compareDogs(troubleMakerComparator); // Compilation ERROR !! Incompatible Types

```

We use variance to solve both these problems.

# Variance

Variance refers to how subtyping between more complex types relates to subtyping between their components. For example, how should a list of `Cats` relate to a list of `Animals`, Or how should a function returning `Cat` relate to a function returning `Animal`



There are four possible variances:  
1. **Covariant** : Preserves ordering of sybtyping relation. i.e. if `A -> B`, then `Box<A> -> Box<B>`  
2. **Contravariant** : Reverses the ordering of subtyping relation. i.e. if `A -> B`, then `Box<B> -> Box<A>`  
3. **Bivariant** : If both of the above applies. i.e. if `A -> B`, then `Box<A> -> Box<B>` & `Box<B> -> Box<A>`  
4. **Invariant / Non-variant** : Neither of above applies.

### Why do we need Covariance & ContraVariance
We need Co-variance and Contra-Variance to solve the above two problems.
if we change the method signature of `printNames` to include a wildcard `?` and make it  `? extends Animal` i.e. 

```java
void printNames(Collection<? extends Animal> animals) {
    ...
}

printNames(myPets); // Compiles cleanly
```
then calling the function `printNames(myPets);` works and type-checks too. `? extends Animal` means any type with an upper bound of Animal is excepted. i.e any type which is a subype of `Animal`. This also makes `List<Dog>` a subtype of `Collection<Animal>` for this method. This is Covariance.


Similarly, to solve for the comparator problem, we change the method `compareDogs ` to include a wildcard and make it `? super Dog`. i.e: 

```java
static void compareDogs(Comparator<? super Dog> comparator) {
    ...
}

compareMyPetDogs(troubleMakerComparator); // Compiles cleanly.

```
Now, calling the function `compareMyPetDogs(troubleMakerComparator);` works fine and makes sense; `troubleMakerComparator` object that we created knows how to compare two Animals and so it can certainly also compare two Dogs. `? super Dog` means any type whose lower bound is Dog i.e. any type which is a supertype of Dog. So `Animal` & `Object` are acceptable types here. This also makes `Comparator<Animal>` a subtype of `Comparator<Dog>` for this method. This is Contravariance.

We can also use wildcards when declaring variables. Ex:

```java
List<Dog> dogs = new ArrayList<Dog>();
dogs.add("PitBull");
dogs.add("Boxer");
List<? extends Animal> animals = dogs;
animals.add(new Cat("Sphynx Cat")); // Compilation ERROR !!
```

Without `? extends`, 4th line would have caused compilation error because `List<Dog> !-> List<Animal>`, but 5th would have worked because `Cat -> Animal`. With `? extends` 4th line compiles because now `List<Dog> -> List<Animal>` but 5th does not compile because you cannot add a `Cat` to a `List<? extends Animal>`,since it might be a list of some other subtype of Animal.

**General Rule of Thumb: If a structure contains elements with a type of the form `? extends E`, we can GET elements out of the structure, but we CAN NOT PUT elements into it.**

Another way to understand: When we `extend`, there is atleast an upper bound type, so we know what type we are getting out of the Box. So GET is allowed, because that's the safest option for a compiler.

Another Example:

```java
public static void addSomeDogsTo(List<? super Dog> someList, int count) {
    if (count == 0)
        return;

    for (int i = 0; i < count; i++)
        someList.add(new Dog("Stray Dog " + i ));

    Animal d = someList.get(0); // Compilation ERROR !! Unknown Type.
}
```

Since we are using `? super`, reading from the list is not possible anymore, because we don't know the type anymore.

**General Rule of Thumb: If a structure contains elements with a type of the form `? super E`, we can PUT elements into the structure, but we CAN NOT GET elements out of it.**

Another way to understand: When we `super`, there is atleast a lower bound type, so we know what's smallest type we can put in the box. So PUT is allowed, because that's the safest option for a compiler.

Note: `Object d = someList.get(0);` works because in Java, everything extends Object. So there is always an upper bound.


### Get and Put Principle

Use an `extends` wildcard when you only get values out of a structure, use a `super` wildcard when you only put values into a structure, and don’t use a wildcard when you both get and put.

### PECS: Producer-Extends, Consumer-Super

This is a mnemonic proposed by Joshua Bloch and is similar to Get and Put Principle. Those object we only read from are called _Producers_, those we only write to are called _Consumers_. The example below shows that destination is where we only read from, and source is where we write. So use `extends` with Producers and `super` with Consumers.

```java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    ...
}
```

# Declaration-site variance in Kotlin/C\# 

Suppose we have a generic interface `Source<T>` that does not have any methods that take `T` as a parameter, only methods that return `T`. Then, it would be perfectly safe to store a reference to an instance of `Source<Dog>` in a variable of type `Source<Animal>` – there are no consumer-methods to call. But Java does not know this, and still prohibits it:

```java

interface Source<T> {
  T nextT();
}

void store(Source<Dog> dogs) {
  Source<Animal> animals = dogs; // Compilation ERROR !!
  // ...
}
```
To fix this we need to add modify `Source<Animal>` to `Source<? extends Animal>` but this is meaningless as there is no value added by the wildcard. Compiler does not know that.

```java
void store(Source<Dog> dogs) {
    Source<? extends Animal> animals = dogs; // Compiles fine.
    // ...
}
```

So, Kotlin/C# has declration-site variance, i.e. annotate type parameter at declaration of the class itself. This is in addition to use-site variance.

```kotlin
interface Source<out T> {
    fun nextT(): T
}

fun demo(dogs: Source<Dog>) {
    val animals: Source<Animal> = dogs // This is OK, since T is an out-parameter
    // ...
}
```

The `Iterable`/`Collection`/`List` types are all declared with `out` parameter in Kotlin. `IEnumerable` is declared as `out` on T in C#.

Java does not have declaration-site variance but could provide something similar using exisiting annotation. i.e. `? extends` or `? super`

Complimentary to `out` there is `in` in Kotlin & C#. 

```kotlin
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}

fun demo(x: Comparable<Animal>) {
    val y: Comparable<Dog> = x // OK!
}
```

**General Rule of Thumb: When a type parameter `T` of a class `C` is declared `out`, it may occur only in out-position in the members of `C`, but in return `C<Derived>` can safely be a subtype of `C<Base>`. Similary, When a type parameter `E` of a class `D` is declared `in`, it may occur only in in-position in the members of `D`, and in return `D<Base>` can safely be subtype of `D<Derived>`**


out, extends, producer, get -> All lie on Covariant side.  
in, super, consumer, put -> All lie on Contravariant side.
