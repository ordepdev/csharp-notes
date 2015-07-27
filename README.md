## Evolution of a basic class

- C# 1: Read-only properties, Weakly typed collections
- C# 2: Private property setters, Strongly typed collections
- C# 3: Automatically implemented properties, Enhanced collection and object initialization
- C# 4: Named arguments for clear constructor and methods

### Evolution of sorting

- C# 1: Weakly typed comparator, No delegate sorting option
- C# 2: Strongly typed comparator, Delegate comparisions, Anonymous methods
- C# 3: Lambda expressions, Extension methods, Option of leaving list unsorted

### Evolution of anonymous methods and lambda expressions

- C# 1: Strong coupling between condition and action, Both are hardcoded
- C# 2: Separate condition from action invoked, Anonymous methods make delegates simple
- C# 3: Lambda expressions make the condition easier to read

### Evolutions of missing data <null>

- C# 1: Choise between extra work mantaining a flag, changing to reference type semantics, or the hack of a magic value
- C# 2/3: Nullable types make the "extra work" option simple, and syntatic sugar improves matters even further
- C# 4: Optional parameters allow simple defaulting

## Delegates

Delegates provide a level of indirection: instead of specifying behavior to be executed immediately, the behavior can somehow be "contained" in an object. That object can then be used like any other, and one operation you can perform with it is to execute the encapsulated action.

- The delegate type needs to be declared
- The code to be executed must be contained in a method
- A delegate instance must be created
- The delegate instance must be invoked

* Delegates encapsulate behavior with a particular return type and set of parameters, similar to a sing-method interface.
* The type signature described by a delegate type declaration determines which methods can be used to create delegate instances, and the signature for invocation.
* Creating a delegate instance requires a method and a target to call the method on.
* Delegate instances are immutable.
* Deletate instances each contain an invocation list - a list of actions.
* Delegate instances can be combined with and removed from each other.
* Events aren't delegate instances - they're just add/remove method pairs (getters/setters).

`delegate void StringProcessor(string input);`  : delegate type
`class X public static void Y(string z)`
`StringProcessor foo = new StringProcessor(X.Y);` : delegate instance
`foo("bar");`

## Type system characteristics

C# 1 is:
* statically typed
* explicit
* safe 

### static typing versus dynamic typing

Each variable is of a particular type, and that type is known at compile time. Only operations that are known for that type are allowed, and this is enforced by the compiler.

### explicit typing versus implicit typing

With explicit typing, the type of every variable must bt explicity stated in the declaration. Implicit typing allows the compiler to infer the type of the variable based on its use.

### type-safe versus type-unsafe

You can't treat one type as if it were another unless there's a genuine conversion available.

## Value types and reference types

Most types in .NET are reference types, and you're likely to create far more reference than value types.

* Array types are reference types | int is value type but int[] is reference type
* Enumerations (enum) are value types
* Delegate types (delegate) are reference types
* Interface types (interface) are reference types | they can be implemented by value types
* String.Empty is not an empty string | it's a reference to an empty string
* Class is a reference type | Struct is a value type
* Value types can't be derived from

You can never change the type of an object - when you perform a simple cast, the runtime just takes a reference, checks whether the object it refers to is a valid object of the desired type, and returns the reference if it's valid or throws an exception otherwise.

### Myths

* structs are lightweight classes
* reference types live on the heap; value types live on the stack
	(a variable’s value lives wherever it’s declared, so if you have a class with an instance variable of type int , that variable’s value for any given object will always be where the rest of the data for the object is - on the heap. Only local variables and method parameters live on the stack)
* objects are passed by reference in C# by default
	(if you pass a variable by reference, the method you're calling can change the value of the caller's variable by changing its parameter value. The value of a reference type variable is the reference, not the object itself. You can change the contents of the object without the parameter itself being passed by reference. if you pass by value a reference to a StringBuilder and try to set it to null, that change wouldn't be seen by the caller, contrary to the myth)

### Boxing and unboxing

The value of a reference type variable is always a reference.
The value of a value type variable is always a value of that type.

int i = 5; => value
object o = i; => reference

A single box or unbox operation is cheap, but if you perform hundreds of thousands of them, you not only have the cost of the operations, but you're also creating a lot of objects, which gives the GC more work to do.

### Summary of value types and reference types

* The value of a reference type expression is a reference, not an object.
* References are like URL's.
* The value of a value expression is the actual data.
* Reference type objects are always on the heap; value types can be either on stack or the heap.
* When a reference type is used as a method parameter, by default is passed by value, but the value itself is a reference.
* Value type values are boxed when reference type behaviour is needed; unboxing is the reverse process.

## New features for > C# 1

### Delegates

* Generics 2
* Delegate instance creation expressions 2
* Anonymous methods 2
* Delegate covariance/contravariance 2
* Lambda expression 3
	(
		Func<int,int,string> func = (x, y) => (x * y).ToString();
		Console.WriteLine(func(5, 20));
	)

### Type System

* Generics 2
* Limited delegate covariance/contravariance 2
* Anonymous types 3
* Implicit typing 3
* Extension methods 3
* Limited generic covariance/contravariance 4
* Dynamic typing 4

### Value Types

* Generics 2
* Nullable types 2

# C# 2

## Features

Generics, Nullable Types, Delegates and Iterators

## Generics

### Why generics are necessary

Casts are bad. Not bad in an almost never do this kind of way but bad in a necessary evil kind of way. They're an indication that you ought to give the compiler more information somehow, and that you're choosing to ask the compiler to trust you at compile time and to generate a check that will run at execution time to keep you honest.

With generics, the compiler can perform more enforcement that leaves less to be checked at execution time and the JIT can treat value types in a particularly clever way that manages to eliminate boxing and unboxing in many situations.

One of the benefits of generics is that more checking is done at compile time, so you're more likely to have working code when it all compiles.

There are two forms of generics in C#: generic types (classes, interfaces, delegates and structures) and generic methods.

A type parameter is a placeholder for a real type and appear in angle brackets within a generic declaration: Dictionary<TKey, TValue>.

Unbound Generic Type: Dictionary<TKey, TValue>
Constructed Type: Dictionary<string, int> - "dictionary of string to int"

Example of a generic method in a nongeneric type
static List<T> MakeList<T>(T first, T second)
List<string> list = MakeList<string>("Item 1", "Item 2");

## Type constraints

* Reference Type constraints - struct RefSample<T> where T : class
	- ensures that the type argument used is a reference type, class, interface, array, delegate...

* Value Type constraints - class ValSample<T> where T : struct
	- ensures that the type argument used is a value type

* Constructor Type constraints - class Sample<T> where T : new()
	- it checks that the type argument used has a parameterless constructor that can be used to create an instance

* Conversion Type constraints - class Sample<T> where T : IComparable<T>
	- lets you specify another type that the type argument must be implicitly convertible
	- Sample<int> is valid beacause int implements IComparable
	- You can specify multiple interfaces, but only on class
	- The type you specify can't be a value type, a sealed class (string) or Object, Enum, ValueType and Delegate

### Combining constraints

* no type can be both a reference type and a value type
* every value type has a parameterless constructor
* the class type constraint must come before the interfaces
* can't specify the same interface more than once
* different type parameters can have different constraints, and they're each introduced with a separate where

## Type inference for type arguments of generic methods

var list = MakeList("Line 1", 1); => fail: the first argument sets list to List<string>

* For each method argument, try to infer some of the type arguments of the generic method
* Check that all the results from the first step are consistent
* Check that all the type parameters needed for the generic method have been inferred

## Implementing generics

static int CompareToDefault<T>(T value)
    where T : IComparable<T>
{
    return value.CompareTo(default(T));
}

* As string is a reference type, the default value is null, and for reference types, everything should be greater than null
* For value types the default value is: int - 0, DateTime - ..MinValue ...
* When a type parameter is unconstrained, you can use the == and != operators, but only to compare a value of that type with null - T value == default(T) => Operator == cannot be applied to operands of type T and T
* When a type parameter is constrained to be a value type, == and != can't be used with it at all - where T : struct => Operator == cannot be applied to operands of type T and <null>
* Comparisions using == and != don't use the overload class methods because the compiler doesn't know what overloads will be available - it's as if the parameters passed in were of type object
* Using EqualityComparer<T> and  Comparer<T> returns an implementation that generally does the right thing for the appropriate type

Type inference example
before: Pair<int,string> pair = new Pair<int,string>(10, "value");
public static class Pair
{
	public static Pair<T1,T2> Of<T1,T2>(T1 first, T2 second)
	{
		return new Pair<T1,T2>(first, second);
	}
}
after: Pair<int,string> pair = Pair.Of(10, "value");

## Limitations of generics in C# and other languages

* Generics don't support variance - a List<string> can't be viewed as an List of its base type, or a List of any of the interfaces it implements

List<Animal> animals = new List<Cat>(); // => invalid
animals.Add(new Turtle()); // => valid

## Summary

* Three main benefits of generics: compile-time type safety, performance and code expressiveness.
* Being able to get the compiler to validate the code early
* Performance is improved most radically when it comes to value types, which no longer need to be boxed and unboxed
* the code is able to express its intention more clearly using generics

