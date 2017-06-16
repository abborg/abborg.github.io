---
layout: post
title: "Nothingness in C# and Visual Basic"
date: 2017-06-09 20:00:00 -0500
category: programming
tags: code CSharp VB Nothing null
published: true
excerpt: "Modern programming languages tend to have two different classes of types: value types and reference types. Value types are simple concepts that can be written as a single value. For instance, some value types are: integers that have values like `3`, booleans that have values like `true`, characters that have values like `A`. Reference types are pointers that _refer_ to an object that is more complex than a singular value. You might have a car object that contains a number of value types combined into a single reference. In certain languages - including C# and VB.Net - there exists the concept of nullable value types. These nullable value types may be `null` prior to being set, but cannot be re-set to `null`. These types allow for variables that may be unset, but once set must have a valid value."
---

* [Value and Reference Types](#value-and-reference-types)
* [Empty Values](#empty-values)
* [Enumerations](#enumerations)
* [Casting Types](#casting-types)
* [Putting them All Together](#putting-them-all-together)
* [A Safe Solution](#a-safe-solution)
* [Sample Code](#sample-code)

<a href="#value-and-reference-types"/>

Value and Reference Types
-------------------------
Modern programming languages tend to have two different classes of types: value types and reference types. Value types are simple concepts that can be written as a single value. For instance, some value types are: integers that have values like `3`, booleans that have values like `true`, characters that have values like `A`. Reference types are pointers that _refer_ to an object that is more complex than a singular value. You might have a car object that contains a number of value types combined into a single reference. In certain languages - including C# and VB.Net - there exists the concept of nullable value types. These nullable value types may be `null`, but also have the properties of value types previously described above. These types allow for variables that may be unset, but can have a single value that can be held in a value type.

<a href="#empty-values"/>

Empty Values
------------
Most - if not all - programming languages have some concept of empty, null, or nil type values for references and/or values. For example, in C the concept of `null` combined two different potential values for types: pointers to memory address `0x0` and values 0, false, and the end of string null character. Other languages however, split these concepts for specificity and improved type-checking by compilers. C# has the `null` keyword along with the `default(Type)` constructor. The `null` keyword is usable on reference types to define that an object has no value or the reference does not refer to anything. The `default` keyword is used to set reference types to `null` and value types to `0` or `false`. In Visual Basic, there is only a single keyword that refers to empty values for both reference and value types: `Nothing`. `Nothing` represents both the concepts of unset references as well as `0` or `false` values, depending on the type of the object.

{% highlight C# %}
// C#
Car bmw; //Declares a reference, but does not refer to any Car object instance; bmw has value of null.
Car volvo = new Car(); // Properly initializes the car object instance and can be referred to later.

int miles; // Declares the integer value type, but assigns no value; miles is set to 0.
int? feet; // Declares the Nullable integer value type, but assigns no value; feet is set to null.
int inches = 12; // Properly declares the integer value type and assigns a value; inches is set to 12.  
	
{% endhighlight %}

{% highlight VB %}
' Visual Basic
Dim ferrari As Car ' Declares a reference, but does not refer to any Car object instance; bmw has value of Nothing.
Dim ford As Car = new Car() ' Properly initializes the car object instance and can be referred to later.

Dim kilometers As Integer ' Declares the integer value type, but assigns no value; kilometers is set to Nothing.
Dim meters As Integer? ' Declares a nullable integer value type, but assigns no value; meters is set to Nothing.
Dim centimeters As Integer = 100 ' Properly declares the integer value type and assigns a value; centimeters is set to 100.

{% endhighlight %}

<a href="#enumerations"/>

Enumerations
------------
One of the most important and useful value types are enumerations. Enumerations are the mapping of a named constant to an integer. They can sometimes be treated like the integers that represent the various named values, but generally they are used to consolidate and standardize a finite set of regularly used values. They can be used to represent different options and are often applied to an option where there is either single or multiple choices that are valid. Enumerations are a value type and as such can only hold non-null values.

{% highlight c# %}
// C#
// Define an enumeration with three constants, explicitly setting the values of the constants.
enum FooType
{
	Foo = 0,
	Bar = 1,
	Baz = 2
} 
	
{% endhighlight %}

{% highlight VB %}
' Visual Basic
' Define an enumeration with three constants, explicitly setting the values of the constants.
Public Enum FizzBuzzType
	Fizz = 0
	Buzz = 1
	FizzBuzz = 2
End Enum

{% endhighlight %}
<a href="#casting-types"/>

Casting Types
-------------
Variables can be converted from one type to another by attempting to convert the values of one object instance to another of a different type. Casting is the process that allows conversion between types. Most primitive types can be cast (relatively) easily between one primitive type and another primitive type. For example in C#, a long integer (64 bit integer) can be implicitly cast to an integer (32 bit integer) by taking the lower 32 bits from the long integer and assigning that value to the integer (this cannot be done implicity in Visual Basic and requires a much more intensive conversion function). However, Object types cannot be cast to one another without an explicitly defined function for converting from type `A` to type `B`. In Visual Basic, the `CType()` and `DirectCast` functions can convert data between types. There are also the `Chr()` and `Asc()` functions to convert to and from characters and character codes.

{% highlight c# %}

// C#
Int64 longInteger = Int64.MaxValue; // Set longInteger to 9,223,372,036,854,775,807 (or in hex 0x7FFFFFFFFFFFFFFF)
Int32 shortInteger = (Int32)longInteger; // Set shortInteger to -1 (or in hex 0xFFFFFFFF)

Integer num = 72; // Set num to 72
Char cha = (Char)num; // Set cha to H

{% endhighlight %}
{% highlight VB %}

' Visual Basic
Dim i As Integer = Int16.MaxValue ' Set i to max value of Int16: 32,767 (or in hex 0x7FFF)
Dim j As Integer = CType(i, Int32) ' Set j to 32,767, but now max value is 2,147,483,647(or in hex 0x7FFFFFFF)
Dim k As Integer = DirectCast(j, Int64)	' Set h to 32,767, but now max value is 9,223,372,036,854,775,807 (or in hex 0x7FFFFFFFFFFFFFFF)

Dim num As Integer = 72 ' Set num to 72
Dim cha As Char = Chr(num) ' Convert Ascii character code to cha to H
Dim val As Integer = Asc(cha) ' Convert character back to Ascii character code 72

{% endhighlight %}

<a href="#putting-them-all-together"/>

Putting them All Together
-------------------------
Things get interesting when combining all of these features and converting between Visual Basic and C#. By calling a function from a Visual Basic library that can return `Nothing`, an unexpected interaction occurs. A sample of this situation can be seen below.

{% highlight VB linenos %}

' Visual Basic library
Module Library
	Function Shared ReturnNothing() As Object
		Return Nothing
	End Function
End Module
{% endhighlight %}

{% highlight C# linenos %}
// C# main function
using Library;
enum SampleEnum
{
	Foo = 0,
	Bar = 1
}
void Main(string[] args)
{
	Object obj = Library.ReturnNothing(); // Set obj to Nothing
	SampleEnum val = (SampleEnum)obj; // Run-time NullReferenceException occurs on casting to a value type
}
{% endhighlight %}

Line 11 is where the issue occurs. Attempting to cast from `Nothing` to a value type seems like an easy casting in C#. As mentioned before, in Visual Basic `Nothing` refers to a value of `0` for value types and `null` for reference types; so why is there a `NullReferenceException` occuring? The `Nothing` keyword and concept only exist in Visual Basic. Once the library function `ReturnNothing` returns the `Nothing` _Object_ from Visual Basic, it is cast to the type used in the function signature. Since the type of the function returning `Nothing` returns an `Object` - or any other reference type for that matter - the `Nothing` is converted to a `null` value. From there, casting it to a non-nullable value type will by definition cause a `NullReferenceException` at run-time.

The issue can be more clearly seen by simplifying the situation; remove the call to Visual Basic and attempt to simply cast `null` to a value type: a compiler error will arise.

{% highlight C# %}
SampleEnum val = (SampleEnum)null; // Compiler error occurs here
{% endhighlight %}

Testing the different possible object types for the variable to accept the returned `Nothing` exposes what .Net is doing when converting between Visual Basic and C#; examine the results when the Visual Basic function returns a non-nullable value type, a nullable value type, and an `Object`.

{% highlight VB linenos %}
Module Library
	Function Shared ReturnNothingAsValue As Integer
		Return Nothing
	End Function
	Function Shared ReturnNothingAsNullable() As Integer?
		Return Nothing
	End Function
	Function Shared ReturnNothingAsObject() As Object
		Return Nothing
	End Function
End Module
{% endhighlight %}
{% highlight C# linenos %}
void Main(string[] args)
{
	int val = Library.ReturnNothingAsValue(); // Sets val to 0
	int? nulval = Library.ReturnNothingAsNullable(); // Sets nulval to null
	Object obj = Library.ReturnNothingAsObject(); //Sets obj to null

}
{% endhighlight %}

<a href="#a-safe-solution"/>

A Safe Solution
---------------
If casting from `Nothing` to C# types, there is a simple and safe way to generically cast between nothing and value types. Using a generic type, a function can be written that will either cast between the types or return the default value for the desired value type. One such generic function can be seen below; though, in order to perfrom conversions from one reference type to another, there must still be an explicitly defined casting function - this generic cast function cannot avoid that necessity.

{% highlight C# linenos %}
T TryGetValueOrDefault<T>(Object val) {
	if(val != null){
		if (!typeof(T).IsSubclassOf(typeof(Enum)) || (typeof(T).IsSubclassOf(typeof(Enum)) && Enum.IsDefined(typeof(T), val)))
			return (T)val;
	}
	return default(T);
}
{% endhighlight %}
This function checks a few things: the parent type of the desired type; if the desired type is an `Enum` type, is there a defined constant for that value; and whether the `val` parameter is null. If `val` is `null`, the default for the desired type is returned. Similarly, if either the desired type is a subclass of `Enum` and there is a defined constant for that `Enum` or it is a non-`Enum` value type, then it is possible to perform a valid cast to the desired type. A safe casting from an arbitrary object to a value type can be done.

<a href="#sample-code"></a>

Sample Code
-----------
If you would like to try messing around with these concepts and see the inner workings of this interaction, you can find some sample code used to test this interaction [here](https://github.com/abborg/blogpost-code/tree/master/6-9-17-CS-vs-VB). 
