Chapter 1 Creating and Destroying objects
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

Builder

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




















