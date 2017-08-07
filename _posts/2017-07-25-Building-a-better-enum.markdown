---
title: "Building a better enum"
date: 2017-07-25 20:00:00 -0500
category: programming
tags: code CSharp C++ Enum
published: true
excerpt: "Recently, I've been working a fair amount with both C# and C++. One thing that I rather enjoyed about C# is the strongly typed enumerations. This one feature of C++ that I find lacking, even into modern C++17 standards. I thought it might be a useful exercise to recreate the versatile and type-safe Enum class from C# in C++."
---


* [Imitation is the sincerest form of flattery](#imitation)
* [Getting started](#getting-started)
* [Comparing apples to slightly nicer apples](#apples-to-csharp-apples)
* [Backwards Compatibility](#backwards-compatibility)
* [Interface & Implementation](#interface-implementation)
* [Inheritance](#inheritance)
* [Other Benefits](#other-benefits)
* [More to come](#more-to-come)

<a href="#imitation"/>

Imitation is the sincerest form of flattery
-------------------------------------------
Recently, I've been working a fair amount with both C# and C++. One thing that I rather enjoyed about C# is the strongly typed enumerations. This one feature of C++ that I find lacking, even into modern C++17 standards. I thought it might be a useful exercise to recreate the versatile and type-safe Enum class from C# in C++.

<a href="#getting-started"/>

Getting started
---------------
I began this project originally as a byproduct of investigating anonymous enumerations in C++. Originally used in C and C++ as a constant that would require no space within the object, it was a way to decrease the overhead of normal static int or char constants that you might otherwise need. Below you can see an example of how one might use these anonymous enumerations.

{% highlight C++ %}
class Color {
    public:
        enum : int { Red, Green, Blue }
};
int r = (int)Color::Red;
{% endhighlight %}

There are a couple of important things to take a look at in this short snippet. First, the enumeration is _not_ of type Color, instead it is a strange Color::<enum> type. Therefore, it is not an instance of a Color object. Secondly, in order to use this enumeration in any useful way, you must explicitly cast the enumeration to its underlying value in order to make comparisons or assign to a variable.

<a href="#apples-to-csharp-apples"/>

Comparing apples and slightly nicer apples
-------------------------
C# Enumerations on the other hand, can be used in a much more elegant way. The System.Enum class comes with a number of features that C++ enums lack - mostly since there is no reflection built-in to C++, which has its own historical baggage. Some of the best features of the System.Enum class are the ability treat the enumerated values as flags, inherited ToString methods, and the IsDefined method. I've found the ToString and IsDefined methods to be extremely useful when attempting to using enumerations in game engines, as they dramatically reduce the code required to produce user-friendly text and provide safety checks on enumerated values, respectively.

<a href="#backwards-compatibility"/>

Backwards Compatibility
-----------------------
One of the main goals with this project was to emulate the C# System.Enum class without losing the syntax of regular enums. Therefore, when possible, the way enumerations can be manipulated and created remains the same. For example, these are some basic operations with standard enums in C++:

{% highlight C++ %}
enum Color {
    RED, BLUE, GREEN
};

Color red = Color::RED;
if(red == Color::BLUE) {
    cout << "I'm blue!" << endl;
}

Color randomColor = GetRandomColor(); // GetRandomColor simply returns red, green, or blue randomly
switch (randomColor) {
    case Color::RED:
        cout << "I'm red" << endl;
        break;
    case Color::BLUE:
        cout << "I'm blue" << endl;
        break;
    case Color::GREEN:
        cout << "I'm green" << endl;
        break;
}

{% endhighlight %}

Through switches, if-statements, assignment, and casting, we see how basic C++ enumerations can be used. We want to replicate that with our new-and-improved C++ classes.

<a href="#interface-implementation"/>

Interface & Implementation
--------------------------
One of the beauties of using a class to implement our new C++ Enum is that we gain the benefits of classes and object oriented programming. For instance, we create an interface seen below to develop a simple means by which we interact with our Enums. This interface also encapsulates the Type enum from the outside scope, meaning that - if it was desired - you could keep the Type and enum protected and provide accessors that can further encapsulate your data.

{% highlight C++ linenos %}
class Enum {
	public:
		typedef int size;
		enum Type : size;
		Type t_;
		std::map<Enum, std::string> n_;
		explicit Enum(int i) : t_((Type)i) {}
		Enum(Type t) : t_(t) {}
		operator Type() const { return t_; }

		template <class T>
		bool IsDefined(T e);
		template <class T>
		std::string GetName(T e);
};
{% endhighlight %}

There are a couple of important lines to note. For example, lines 3-4 and lines 7-9. We'll take a look at each line to see what they're for.

Starting with lines 3 and 4, we define the type size to be an alias of int, then we use size to define the underlying type of our inner enum. While it may seem convoluted, this means that we can define multiple classes with different underlying types. This could be useful to keep space limited if you know you only have a few values (some compilers might ignore your declaration of an underlying type smaller than an int, but that's neither here nor there). We can also define a macro that takes different typenames to create a variety of Enum parent classes.

{% highlight C++ %}
typedef int size;
enum Type : size;
{% endhighlight %}

Next, we'll move to lines 7, 8, and 9. Line 7 has a keyword that I had not seen until I started working on this project. The 'explicit' keyword informs the compiler that this specific conversion operator should only be used when it is explicitly invoked. Otherwise, it should be treated as if there was no conversion between the two types. 

{% highlight C++ %}
explicit Enum(int i) : t_((Type)i) {}
{% endhighlight %}

This is extremely useful when working with enumerations as they can be treacherous when using them alongside integers or converting between an enum and its underlying type. The compiler is allowed to use additional operator whenever one has been explicitly invoked. While this may not seem like an issue, consider the following:

{% highlight C++ %}
int i = -1;
int j = 1;
if(i + j) {
	cout << "It's zero" << endl;
}
{% endhighlight %}

Here, we see an addition operator being invoked to add two integers. Since if-statements take boolean values, the compiler must now convert from an integer value to a boolean. The compiler is allowed to do this conversion as an additional operation. However, this is being done implicitly, so to prevent this and avoid the [type-safe boolean issue](TODOcheck), we must declare our conversion operator explicit. The exlicit keyword can be used on conversion operators as well as constructors. In our case we use it with a Enum constructor from an int to avoid implicit construction of undefined Enum values.

{% highlight c++ %}
Enum(Type t) : t_(t) {}
operator Type() const { return t_; }
{% endhighlight %}

On these next two lines, we see a Enum constructor from a type and an implicit conversion operator. The constructor simply takes in a value of the same type as the underlying enum and set the value of `t_` to that. The user-defined conversion operator is declared on the second line of this snippet. It is defined by declaring an operator, a type, and a code block that returns an object of that type. Since the explict keyword was not expressed, the compiler is allowed to use this conversion operator for both implicit and explicit conversions. These two lines are useful for converting two and from to and from our Type enum without calling the explicit conversion operator or constructor by casting them.

<a href="#inheritance"/>

Inheritance
-----------
As mentioned before, one of the benefits of utilizing a class to implement this Enum is that we gain the benefits of working with objects. Some of those benefits (member defined operators, data encapsulation) were discussed previously, but we will now look at the importance of Inheritance to this implementation. For example, we can create a child class of our Enum class. Within this class we will define the enumerated values and map our values to strings for our GetName method.

{% highlight c++ linenos %}
class Color : public Enum {
	public:
		enum Type : size {
			RED,
			YELLOW,
			BLUE,
			ORANGE,
			GREEN
		};
		explicit Color(size s) : Color(Color::Type(s)) {}
		Color(Enum e) : Enum(e){}
		Color(Type t) : Enum(t) {}
		operator Enum::Type() const { return t_; }
		static bool IsDefined(Color c);
		static std::string GetName(Color c);
	private:
		static std::map<Color, std::string> n_;
};
{% endhighlight %}

Here, we define a Color class. In it, we declare five enumerated values for colors. We also see similar lines to that of the parent class, and we can take a look at those more closely.

In the following snippet (also, lines 10-13 in the longer snippet above), we see similar conversion operators as well as constructors. One thing to note is that we use the size type rather than integer for an explicit constructor. This way, we can inherit the defined type in the Enum class and create multiple different parent Enum classes with different enum sizes, but all refer to that size with the same typename. This explicit Color constructor takes in a size (aka int) and passes it to the Color constructor for Color::Type. At the same time, it constructs a Color::Type from the size variable the constructor was called with. We do this so that we can explicitly cast integers to our enum, which is part of our desire to keep the syntax of this Enum class the same as normal enums in C++;

The next two lines, we see simple implicit constructors that will consume an Enum or a Type. This may be confusing but this Type is a Color::Type rather than an Enum::Type which is why we need both constructors.

Finally, the implicit conversion operator to the parent Enum::Type type is necessary to utilize constructors of other child types of the Enum class. TODO::check this

{% highlight c++ %}
explicit Color(size s) : Color(Color::Type(s)) {}
Color(Enum e) : Enum(e){}
Color(Type t) : Enum(t) {}
operator Enum::Type() const { return t_; }
{% endhighlight %}

We defined a couple of methods that we override in these child classes. These methods are staticly defined so that we can call this methods outside of an instance of these classes. 

{% highlight c++ %}
static bool IsDefined(Color c);
static std::string GetName(Color c);
{% endhighlight %}

We privately define the map of types to strings that we will be using in our GetName function. These are defined privately to prevent externally accessing and altering the values of our enum names. They are also defined as static so that the values are initialized as soon as the class is utilized and does not require a specific instance of the class.
{% highlight c++ %}
static std::map<Color, std::string> n_;
{% endhighlight %}

<a href="#other-benefits"/>

Other Benefits
--------------
There are a few benefits of utilizing object oriented programming when implementing this new Enum type.

Since these objects different classes, they are safe from accidental conversion between constants of different types. With integer constants, there is no safety when moving between constants of (what should be) different enumerations. The built-in enum and enum class types are type-safe between each other but come with their own issues. Enumerations defined with the enum type are and they cannot be converted between without casting. However, a variable cannot be used in combination with a resolution of an enumeration 

As these types are implemented as classes, we can define member methods and operators. These member operators can be utilized to perform bitwise operations as if the enumerations encapsulated in the Enum class are bit flags rather than exclusive values. This is similar to the FlagAttribute attribute on the C# Enum class.

Namespace pollution can also be an issue with enums as they must be global in order to be used in many places. This also means that you cannot have two enums with the same name globally. By encapsulating the similarly named enumerations into multiple different classes that represent the overarching data contained within the enumeration, we can limit the pollution to the namespace or potential collisions with other enumerations.

We can further expand on this concept of enumerations within a class to create multiple enumerations within one class. This may be useful for a 'Defined Contants' class, where there are multiple enumerated types contained within a single static class so that they can all be included with a single include.

<a href="#more-to-come"/>

More to come
------------
There are still other features of this enum class that I haven't mentioned here or will be developed later. If you would like to see the source and some example code for this project, you can check the repo for this post [here, on github](https://github.com/abborg/blogpost-code/7-25-17-Emulating-C#-Enum/CSharp-Enum). Feel free to create issues for features or use this source in your own projects.