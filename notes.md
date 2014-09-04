Chapter 2. Creating and Destroying objects
=========================================

1. Static factory method
------------------------

Advantages over constructors

* can have names
  * if constructors have many parameters, this can improve readabiilty of code
* don't have to return brand new object
  * useful for instance-controlled classes: singletons, immutable classes, non-instantiable classes etc
* can return object of subtype of declared return type
  * interface-based frameworks, return types are interfaces and the actual implementation is not public class
  * interfaces can't have static classes, by convetion the factory method for class Type is in class called Types
    * both previous bullets are used in Java Collections Framework: Array and Arrays, Collection and Collections
  * service provider framework pattern

Disadvantages

* no constructors means that class can't be subclassed
* more difficult to find in doc, unlike constructors, there is no special treating by javadoc

2. Builder
----------

Used if the class has many (optional) parameters. Constructors and factory methods don't scale well in such cases. Other possibilities: 
* Telescoping Constructor Pattern
 * many constructors with growing number of parameters
 * don't scale well -> hard to write client code and hard to read it
* JavaBeans pattern 
 * parameter-less constructor plus setters
 * you can't make the object immutable
 * the object is in inconsistent state
 * more difficult to be thead-safe

* Builder

Create static member class (builder) with the same parameters as buildee class. Create private factory method or constructor in buildee class that expects builder instance and extracts all the information from builder when creating the buildee instance. Builder contains the build() method that uses this private factory method or constructor.

Buildee {int param1, param2..., param100; public static class Builder {int param1...100 = 0; public Builder(){} public Builder setParamN(int val) {paramN=val;} public Buildee build() {return new Buildee(this);} }// end of Builder
private Buildee (Builder b) {paramN = b.paramN } }// end of Buildee

3. Singleton
------------

Preferred way is to use one-element Enum
public enum Elvis { INSTANCE; /* members of class */ }

4. Noninstantiability
---------------------

Using private parameterless constructor, can throw AssertionError() in the body. Useful for utility classes with all static methods, or Arrays, Collections -like classes.

5. Avoid creating unnecessary objects
-------------------------------------

Always reuse immutable objects. Typically String s = new String("my string"); is stupid idea in most cases.
Use static factory method is available, they can return existing instance, the constructors always create new instance,
for instance, use Boolean.valueOf(String) instead new Boolean(String)
You can reuse mutable object too, if the logic of your program don't need to change that objects.
Prefere primitives over boxed primitives if possible (e.g.: use long instead of Long)

6. Eliminate obsolete object references
---------------------------------------

Ensure the garbage collector can remove the unnecessary objects. Explicitely null the reference if needed. The best way is to have references in narrowest possible score.

7. Avoid finalizers
-------------------

Unpredictable, don't need to be invoked at once, don't need to be invoked at all, exceptions thrown in finalize() are ignored. Instead use explicit termination method. Like Inputstream/java.sql.Connection close() or java.util.Timer cancel(). Disadvantage is that client code is responsible for termination method calling. Usually combined with try-finally construct.

If using finalize() in subclass and the parent class has finalize() too, you have to explicitely call super.finalize() in order to the subclass' finailze() call the perent's finalize(). There is the finalizer guardian pattern how to run parent's finalize() even if the programmer forgets to call super.finalize() in subclass' finalize().


Chapter 3. Methods common to all objects
========================================

8. Obey the equals contract
---------------------------

Only if the instances represent some "values", not active entities (e.g. Random class). Reflexivity, Symmetry, Transitivity. Usually broken if you trying to implement interoperability of equals methods in classes and subclasses.
E.g. You have Point p and PointWithColor pwc class instances. p.equals(pwc) is true since the coordinates are the same, but pws.equals(p) is false because equals in PointWithColor takes color into account.

"There is no way to extend an instantiable class and add a value component while preserving the equals contract."

Hints to create equals method:
* use == at first
* use instanceOf (this returns false if first operand is null thus no explicit null check is required)
* cast argument
* do the equality test on all the significant fields
* equals should depend on stable data the instance contains
 * if class uses some other resources (network connection, database access, related fields should not be taken into consideration of equals method
* override the hashCode method as well

9. Always override hashCode when you override equals
----------------------------------------------------

Important for hash-based solutions, typically subclasses of Collection, Map classes. The genaral approach how to tackle the hashCode function is to recursively compute hashes for significant fields and combine them together.
* store some number, usually a prime in a variable that will store our hash, let's call it result (int result = 17)
* create variable that will store the hashes of fields, let's call it fieldHash, fill it using rules below
* boolean -> FALSE 0, TRUE 1
* byt, char, short, int -> just cast to int
* long f -> (int) (f ^ (f >>> 32)); // ^ XOR, >>> unsigned bit right shift
* float f -> Float.floatToIntBits(f);
* double f -> Double.doubleToLongBits(f) and hash the resulting long
* object reference -> use it's own hashCode function
* Arrays or Collections -> compute hashes of all items and combine then together as shown in next bullet
* null -> arbitrary constant, traditionally just zero (0)
* result = 31  * result + fieldHash

10. Always override toString
----------------------------

* useful for diagnosis -> easy log messages
* useful for value classes -> easy to see the content
* should return info about all significant fields
* every information accessible via toString should be available via getters or some other methods

11. Override clone judiciously
------------------------------

* Cloneable is marker interface, its presence changes the default behavior of Object's protected clone() method
* typically you start with super.clone()
* clone() creates instance without using constructors!, although constructors can be used in clone() body
* if class is final, the clone() can ever return the instance created via constructor
* in non-final classes, you should return an instance returned by super.clone() thus eventually returned by Object's clone()
* clone() is not compatible with using final fields reffering to mutable objects
* it can be better to use copy constructor (e.g. new String("string"); or copy static factory insead copy()
* copy constructor/static factory method can be interface based (for cloning HashSet to TreeSet: new TreeSet(hashSet))

12. Consider implementing Comparable
------------------------------------

* if you have "value" class and it has some "natural ordering"
* once comparable, many additional features are available (sorting, max, min...), see Collections framework
* recommended that equals() is consistent with compareTo(), but not required, for instance BigInteger class
* simpler that equals because interoperability between subclasses is not expected, nor recommended; but possible

Chapter 4. Classes and Interfaces
========================================

13. Minimize the accessibility of classes and members
-----------------------------------------------------

* properly hide internal implementation and expose API only
 * better testable, reusable, independently optimized, perf tuned, internals can be replaced in next versions... 
* as inaccessible as possible
* beware, protected is part of API, only private and package-private are internals
* public static final fields should reference immutable object only, otherwise it is not a real constant

14. In public classes use accessor methods, not public fields
-------------------------------------------------------------

otherwie
* can't change internal representation without changing API
* can't enforce invariants
* can't take auxilliary action when accessing fields

15. Minimize mutability
-----------------------

Total Immutability
* no method that modify the state
* not subclassable (extendable) using final keyword or more flexible static factory method
* all fields final
* all fields private so other classes can't access mutable inner objects
* methods return new object, not the 'this' object (like all String methods)
* inherently thread-safe, no need for sync; fully shareable between threads

Narow the space of possible states as much as possible
* the mutable twin of the immutable class can be provided (e.g. String vs StringBuilder)

16. Favor composition over inheritance
--------------------------------------

* inheritance violates encapsulation, subclasses depend on proper functionality of super classes
 * super class implementation can be changed any time and possibly break subclass behavior
* use composition, especially if the class implements some interface
 * create forwarding (delegating) class that implements the interface which will have the interface as private field
 * create instrumented class extends the forwarding class and overrides desired methods of forwarding class

17. Design and document for inheritance or else prohibit it
-----------------------------------------------------------

* document self-use (every circumstance the overridable method can be invoked under)
* sometimes you need to expose some internals so they can be overriden due to perf optimizations
* constructors can't use overridable methods (not properly overriden if sublcass calls the super constructor)
* Serializable, Clonable -> readObject() and clone() can't call overridable methods too (they are like constructors)
* never allow inheritance for immutable classes, the wrapper pattern works better in this case

18. Prefer interfaces to abstract classes
-----------------------------------------
* existing classes can be retrofitted to implement new interface, abstract classes not
 * not always you have good place where to insert abstract class into class hierarchy
* interfaces are ideal for implementing mixins
* interfaces allow to use wrapper pattern
* having interface you can always create abstract class anyway (skeletal implementation)
 * programmers can choose to use it or not
 * if you don't want use skeletal implementation but some of the pieces are useful:
  * create inner private class extending skeletal implementation
* think hard, interface can't be changed in future releases, but you can add method to abstract class

19. Use interfaces only to define types
---------------------------------------

* don't use marker interfaces (with no methods), it not a good concept alhough it is seen quite often in JDK classes
* no constant interfaces (with static final field definitions only)
 * use non-instantiable utility class and static imports instead

20. Prefer class hierarchies to tagged classes
----------------------------------------------

* tagged class -> contains field that further determine it's "type"
 * class Figure { enum Shape {RECTANGLE, CIRCLE}; double radius, lenght, width; ... }
* replace by abstract class with methods that don't depend on tagged field and the subclasses for every tagged value

21. Use function object to represent strategies
------------------------------------------------
!!! will be way different with lambda expressions introduced in Java8 !!!

* java doesn't allow to transmit function, use object references instead
* Strategy class: stateless class with method representing the strategy that operates on object passing in as parameters
* Stategy interface: for strategy class (e.g. Comparator<T> is strategy interface, any implementation is strategy
* such strategy can be used, Comparator is used in Java Collection Framework (sort, max, ...)
* create private strategy class and public host class having public interface of strategy class if you don't want to expose strategy class
* 
22. Favor static member classes over non-static
-----------------------------------------------

* static member class is defined inside enclosing class and has access even to private members of enclosing class
* non-static one is associated to specific instance of enclosing class, thus every enclosing instance has its own instance of non-static, which is inefficient if you don't need it

Chapter 5. Generics
========================================

23. Don't use raw types in new code
-----------------------------------

* always use generics, there are two exceptions:
 * instanceOf operator if (someInstance instanceof List) {...}
 * class literals: List.class
* List<String> is subtype of List, but not of List<Object>; generics provide type safety
* List<?> is better than List because you can't add new element into List<?> but null
 * if you don't know or care, use List<?>, e.g. if (set instanceof Set) { Set<?> set1 = (Set<?>) set; ... }

24. Eliminate unchecked warnings
--------------------------------

* always do if possible
* it not, add @SuppressWarning but to the smallest scope possible and comment why it can't be eliminated
* it's worth to introduce variable just because it enables us to use smallest scope for @SupressWarning annotation


25. Prefer lists to arrays
--------------------------

* generics lists are more type-safe, type errors revealed in compile-time, not runtime
* there are many problems with combining arrays and list
 * varargs
 * creating array from list in general not possible

26. Favor generics types
-------------------------

* generificatio is quite straightforward, problems can be with arrays
* you can retype Object arary to (E[]), but you should add @SuppressWarning and make sure that the usage is type-safe

27. Favor generics methods
--------------------------

* type-safer
* client doesn't need to cast output
* it is possible to generify the method declarations without breaking existing clients

28. Use bounded wildcard type to increase API flexibility
---------------------------------------------------------

* in method declarations using just E can be too restrictive
* use "? extends E " for producers
 * if you produce value for List<E> (push), your value should be of type "? extend E" so it can be added to list safely
* use "? super E" for consumers
 * if you consume value from List<E> (pull), you can safely store it in variable of type "? super E"
  * this is less restrictive == more flexisble than using just E
* avoid using wildcard types as return types
 * it forces clients to deal with it in client code
* use Comparable<? super T> in preference to Comparable<T> (same for Comparator class)
* if type parameter (e.g. E) appears only once in method declaration, replace it by wildcard ("?")
 * if you have compile errors, (e.g. can't insert item to List<?>), create private helper method that uses E
 * but in API (public) methods, use wildcard if possible

29. Consider typesafe heterogeneous containers
----------------------------------------------

* approach that enables storing instances of different classes in one container(map) in typesafe manner
* using wrapper with private map, the key is parametrized: Map<Class<T>, Object>
* the key holds the information about compile-time type after the instance (map value) is stored as Object
* when using wrapper get, the instance (map value) can be casted properly
* usage of <Class<T>> as a holder for compile-time type information is called "type token"

Chapter 6. Enums and Annotations
================================

30. Use Enums instead of int constants
--------------------------------------

* use if implementing some "naturally enumerated value" like planets, seasons of year, ....
* use if implementing values that are all known in compile-time (choices on a menu, miscellaneous flags, ...)
* more convenient (readability, type-safety, ...)
* provide implementation of Object's method, Comparable, Serializable
* can contain data, methods, constructors, ...
* can implement interfaces, can have abstract methods
* immutable (all fields should be final)
* if you override toString(), consider also implementing fromString(String) method that returns the Enum instance
* each instance can have specific behaviour
 * try to avoid swich statements
 * create inner private enum (strategy enum) and pass it to outer enum via constructor instead

31. Use instance fields instead of ordinals
-------------------------------------------

* ordinal() -> very few valid use cases for application developers
* never derive value associated with enum from its ordinal
 * create a field and initialize it via constructor instead

32. Use EnumSet instead of bit fields
-------------------------------------

* if using enum primarily in set, using bit fields we can perform set operations effectively via bit operations
* EnumSet is designed to replace this pattern, it uses bit vector internally
* bit: apply(STYLE-ONE | STYLE-TWO), enumset: apply(Enumset.of(Style.ONE, Style.TWO)

33. Use EnumMap instead of ordinal indexing
-------------------------------------------

* never use ordinal as indices or keys
* use EnumMap instead -> it is optimized for Enums using as keys
* use nested EnumMap if mapping from two or more Enums is required

34. Emulate extensible Enums via interfaces
-------------------------------------------

* enums can implement interface
* if API uses interface strictly, you can create other enum implementing the interface
 * this enum can be used in addition to original one

35. Prefer annotations to naming patterns
-----------------------------------------
* naming pattern -> if client code (some framework) derives some meta information about method from method name
 * e.g. JUnit considered all methods starting with test...() to be test methods
* now there is no reason for this, use annotations instead
 * the framework can via reflection access the annotations and do some processing
 * annotations can have fields, so you can store there values required for the processing
* use predefined annotation types provided by the Java Platform (e.g. @Override)

36. Consistently use @Override annotation
-----------------------------------------

* you can discover errors at compile time
 * both unintentional overriding and not-overriding when desired
* use consistently everytime you believe you overriding a superclass declaration
 * not needed if overiding abstract method, compiler will generate error anyway

37. Use marker interfaces (MI) to define types
-----------------------------------------

* advantages over marker annotations (MA)
 * MI define types so you can use them as input/output parameters in method declarations
 * can be targeted more precisely, e.g. only on specific subclasses
* advantages of MA
 * more information can be added after the release
 * can be added to any program element, not only on class level

38. Check parameters for validity
---------------------------------

* method can fail quickly and cleanly with approprate exception
* for public methods use @throws tag in javadoc
* for private use assert instead of exceptions
* particulary important for parameters that are stored for later use
* can be skipped if check is expensive or done implicitly during method execution

39. Make defensive copies when needed
-------------------------------------

* program defensively with assumption that clients do everything to destroy your code and its invariants
* make defensive copy of each mutable parameter to the constructor
* defensive copies created before validity check
* don't use clone method, it doesn't have to return instance of the class but some untrusted subclass
 * can be overriden by untrusted parties 
* return defensive copies of mutable internal fields (in getters)
 * can use clone method here


40. Design method signatures carefully
--------------------------------------

* choose method name carefully, obey the standard naming conventions
* don't try to provide method for everything, the client can compose provided methods for more complex functionality
* avoid long parameter list, especially long sequences of indentically typed parameters
 * split to more methods
 * use helper classes that will group some of the parameters
 * adapt Builder patern (item 2) for the method with many parameters
 * favor interfaces over classes
 * prefer two-element enum to boolean parameters

41. Use overloading judiciously
-------------------------------
* choice of which method is used is made at compile-time (based on compile type, declared class/interface)
 * different behaviour that overriding which is based on runtime-type (real used class/implementation of interface)
* save conservative approach: never overload method with the same number of parameters
 * use writeInt, writeLong, writeBool instead of overloading write method
* at least don't overload method with the parameters that can be cast to each other (are related)
 * beware of auto-(un)boxing, e.g. remove(E) and remote(int) is confusing
 * you can do this if the overloading methods behave equally (e.g. use the same method internally)

42: Use varargs judiciously
--------------------------------

* use varargs only when a call really operates on a variable-length sequence of values
* if one or more params are required, use first param normally and then varargs

43: Return empty arrays or collections, not nulls
------------------------------------------------------

* client code doesn't have to perform null check
* creating empty array/collection is not a performance problem
 * Collections.emptyList() for efficient creating and returning empty collection

44: Write doc comments for all exposed API elements
---------------------------------------------------

* precede every exported class, interface, constructor, method, and field declaration with a doc comment
* The doc comment for a method should describe succinctly the contract between the method and its client
 * summary description, finished by '.' (dot) 
 * preconditions, postconditions, side effects
 * param, return tags (followed by noun phrase), throws tag (followed by if...), no '.' at the end
 * no two members or constructors in a class or interface should have the same summary description

Chapter 8 General Programming
=============================

45: Minimize the scope of local variables
-----------------------------------------

* declare them where they are first used
* nearly every local variable declaration should contain an initializer, if not, you probably can define it later
* prefer for loops to while loops (because of local variable i doesn't escape the scope of the loop)
* keep methods small and focused
 



reading: item 62.
