# C\# Advanced Topics

Table of Contents
=================

   * [C# Advanced Topics](#c-advanced-topics)
   * [Table of Contents](#table-of-contents)
      * [Generics](#generics)
         * [Different type of constraints](#different-type-of-constraints)
         * [IEnumerable vs ICollection vs IList](#ienumerable-vs-icollection-vs-ilist)
            * [IEnumerable](#ienumerable)
            * [ICollection](#icollection)
            * [IList](#ilist)
      * [Delegates](#delegates)
         * [Action, Func and Predicate](#action-func-and-predicate)
            * [Action](#action)
            * [Func](#func)
            * [Predicate](#predicate)
         * [Interfaces vs Delegates](#interfaces-vs-delegates)
      * [Lambda Expressions](#lambda-expressions)
      * [Events](#events)
      * [Extension Methods](#extension-methods)
      * [LINQ](#linq)
      * [Nullable Types](#nullable-types)
      * [Dynamic](#dynamic)
      * [Exception Handling](#exception-handling)
      * [Asynchronous Programming with Async/Await](#asynchronous-programming-with-asyncawait)
      * [Polymorphism](#polymorphism)
         * [Method hiding](#method-hiding)
         * [Method overriding](#method-overriding)
         * [Method overloading](#method-overloading)
      * [Access Modifiers](#access-modifiers)
      * [yield return](#yield-return)
      * [Value types vs Reference types](#value-types-vs-reference-types)
         * [Value type](#value-type)
         * [Reference type](#reference-type)

Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

## Generics
Generics were added to C# quite early in order to make it possible to reuse the same classes in multiple places. For instance, instead of having to create a class called BookList, CarList etc. for every specific type of list that we want, we can now use a generic type of list.

Below are some example usages of creating generics. The consumer of the methods/classes specifies what type T should be. We also have a couple of constraints, for instance we have two for GenericList: `where T: class, new()`. These constraints specify that the consumer type T must be a class and that it must have a parameterless constructor.

```cs
// Generic List
public class GenericList<T> : where T: class, new() {
    public void Add(T value) {
        T tmp = new T(); // Would not be able to do this if we do not have the new() constraint
    }

    public T this[int index] {
        get { throw new NotImplementedException(); }
    }
}

// Generic dictionary
public class GenericDictionary<K, V> {
    public void Add(K key, V value) {

    }
}

// Generic Max method
public T Max<T>(T a, T b) where T : IComparable {
    return a.CompareTo(b) > 0 ? a : b; // Would not have access to CompareTo method without the IComparable constraint
}

// Generic Nullable (A struct similar to this is already part of System.Nullable)
public class Nullable<T> where T : struct {
    private object _value;

    public Nullable(){}

    public Nullable(T value) {
        _value = value;
    }

    public bool HasValue {
        get { return _value != null; }
    }

    public T GetValueOrDefault() {
        if (HasValue) return (T) _value;

        return default(T); // Return the default value of the value type (0 for int etc.)
    }
}
```

> In most cases you will be using Generics and not create your own.

In .NET, the generic predefined classes are located in *System.Collections.Generic.\**.

### Different type of constraints
* `where T : IComparable` - Constraint T to an interface
* `where T : Product` - The consumer must be a Product or any class that derives from Product (a subclass of Product)
* `where T : struct` - T must be a value type (directly store the value in memory like int)
* `where T : class` - T must be a class
* `where T : new()` - T must have a default constructor

### IEnumerable vs ICollection vs IList
These three generics are very common. The difference between them is the following:  

#### IEnumerable
An IEnumerable is a list or a container which can hold some items. You can iterate through each element in the IEnumerable. You can not edit the items like adding, deleting, updating, etc. instead you just use a container to contain a list of items. It is the most basic type of list container.  

All you get in an IEnumerable is an enumerator that helps in iterating over the elements. An IEnumerable does not hold even the count of the items in the list, instead, you have to iterate over the elements to get the count of items.
An IEnumerable supports filtering elements using where clause.

#### ICollection
ICollection is another type of collection, which derives from IEnumerable and extends it’s functionality to add, remove, update element in the list. ICollection also holds the count of elements in it and we does not need to iterate over all elements to get total number of elements.

#### IList
IList extends ICollection. An IList can perform all operations combined from IEnumerable and ICollection, and some more operations like inserting or removing an element in the middle of a list.

## Delegates
Delegate definition:
* An object that knows how to call a method (or a group of methods)
* A reference to a function

Delegates makes it possible to design extensible and flexible applications (e.g. frameworks). If we look at the code example below, we create a program for processing photos and adding different filters to the processing. Instead of having a set of filters that are always applied (the code that is commented out), we can use delegates and instead make the user apply the filters that they want when processing a photo. This also allow the users to create and apply their own filters aslong as it meets the delegate's function signature.

> Delegates are very similar to using the Decorator pattern!

```cs
namespace Delegates {
    class Photo {
        public static Photo Load(string path) {
            return new Photo();
        }

        public void Save() {}
    }

    class PhotoFilters {
        public void ApplyBrightness(Photo photo) { Console.WriteLine("Apply brightness") }
        public void ApplyContrast(Photo photo) { Console.WriteLine("Apply contrast") }
        public void Resize(Photo photo) { Console.WriteLine("Resize photo") }
    }

    class PhotoProcessor {
        public delegate void PhotoFilterHandler(Photo photo);
        
        public void Process(string path, PhotoFilterHandler filterHandler) {
            var photo = Photo.Load(path);

            filterHandler(photo);

            // BEFORE USING DELEGATES
            // var filters = new PhotoFilters();
            // filters.ApplyBrightness(photo);
            // filters.ApplyContrast(photo);
            // filters.Resize(photo);

            photo.Save();
        }
    }

    class Program {
        static void Main(string[] args) {
            var processor = new PhotoProcessor();
            var filters = new PhotoFilters();
            PhotoProcessor.PhotoFilterHandler filterHandler = filters.ApplyBrightness;
            filterHandler += filters.ApplyContrast;
            filterHandler += RemoveRedEyeFilter;

            processor.Process("photo.jpg", filterHandler);
        }

        static void RemoveRedEyeFilter(Photo photo) { Console.WriteLine("Apply RemoveRedEye") }
    }
}
```

### Action, Func and Predicate
Action, Func and Predicate are two predefined delegates that are shipped with the .NET framework.

#### Action
Action points to a method that accepts one or more arguments and returns void.

The code example below prints "Hello!!!" to the console using the Action delegate.

```cs
static void Main(string[] args)
{
    Action<string> action = new Action<string>(Display);
    action("Hello!!!");
    Console.Read();
}

static void Display(string message)
{
    Console.WriteLine(message);
}
```

Below is the photo processing example using Actions instead:

```cs
class PhotoProcessor {
    public delegate void PhotoFilterHandler(Photo photo);
    
    public void Process(string path, Action<Photo> filterHandler) {
        var photo = Photo.Load(path);

        filterHandler(photo);

        photo.Save();
    }
}

class Program {
    static void Main(string[] args) {
        var processor = new PhotoProcessor();
        var filters = new PhotoFilters();
        Action<Photo> filterHandler = filters.ApplyBrightness;
        filterHandler += filters.ApplyContrast;
        filterHandler += RemoveRedEyeFilter;

        processor.Process("photo.jpg", filterHandler);
    }

    static void RemoveRedEyeFilter(Photo photo) { Console.WriteLine("Apply RemoveRedEye") }
}
```

#### Func
Func points to a method that accepts one or more arguments and returns a value.

The following code snippet illustrates how you can use a Func delegate in C#. It prints the value of Hra (calculated as 40% of basic salary). The basic salary is passed to it as an argument.

```cs
static void Main(string[] args)
{
    Func<int, double> func = new Func<int, double>(CalculateHra);
    Console.WriteLine(func(50000));
    Console.Read();
}

static double CalculateHra(int basic)
{
    return (double)(basic * .4);
}
```

> Note that the second parameter in the declaration of the Func delegate in the code snippet represents the return type of the method to which the delegate would point. In this example, the calculated Hra value is returned as double.


#### Predicate
A Predicate delegate is typically used to search items in a collection or a set of data.

The code example below creates a Predicate delegate filter that is used to filter out the persons with Id = 1. The output in this case would be "Jacob".

```cs
class Customer
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
}

class Program {
    static void Main(string[] args)
    {
        List<Customer> custList = new List<Customer>();
        custList.Add(new Customer { Id = 1, FirstName = "Jacob", LastName = "Hallman" });
        custList.Add(new Customer { Id = 2, FirstName = "Elon", LastName = "Musk" });
        Predicate<Customer> firstCustomer = x => x.Id == 1;
        Customer customer = custList.Find(firstCustomer);
        Console.WriteLine(customer.FirstName);
        Console.Read(); 
    }
}
```

> Note that Predicate<T> is basically equivalent to Func<T,bool>.

### Interfaces vs Delegates
An alternative to delegates is using interfaces, how do we decide which to use?

**Use a delegate when**
* An eventing design pattern is used.
* The caller doesn't need to access other properties or methods on the object implementing the method.
* The interface defines only a single method.
* Multicast capability is needed (combining filters for instance).
* The subscriber needs to implement the interface multiple times.

Below are two code examples, one using interfaces and one using delegates:

```cs
interface IMyInterface    
{    
    int MySum(int a, int b);    
}    
  
public class MyClassTest : IMyInterface    
{    
    public int MySum(int a, int b)    
    {    
        return a + b;    
    }    
}    
```

```cs
public delegate int MyDelegate(int a, int b);    
public class MyClass    
{    
    public static int Addition(int a, int b)    
    {    
        return a + b;    
    }    
}    
```

Comparing usage:

```cs
static void Main() 
{
    // Interface:
    IMyInterface myClassTest = new MyClassTest();    
    Console.WriteLine(myClassTest.MySum(10, 20));    
    
    // Delegates:
    MyDelegate myDelegate = MyClass.Addition;    
    Console.WriteLine(myDelegate(10, 20));    
}
```

So, Delegate made it smaller. But, it can be written in more optimized and smaller way like the following: 

```cs
static void Main(string[] args)    
{    
    Func<int, int, int> operation = (a, b) => a + b;    
    Console.WriteLine(operation(10, 20));    
    Console.ReadKey();    
}   
```

There are so many differences between Delegate and Event. But, I will explain one practical difference between them.

The main difference is that different delegate instances can be provided for the same delegate from the same class. But, it did not happen with Interface.

We can see this by looking at the following example comparison:

```cs
public delegate int MyDelegate(int a, int b);    
public class MyClassResult    
{    
    // Below All methods can be used to implement the MyDelegate Delegate.    
    void M1(int MyDelegate) { }    
    void M2(int MyDelegate) { }    
    void M3(int MyDelegate) { }    
    void M4(int MyDelegate) { }    
}   
```

```cs
interface IMyInterfaceExample    
{    
    void MyResult(int a, int b);    
}    
   
public class MyClassExample : IMyInterfaceExample    
{    
    public void MyResult(int a, int b)    
    {    
        // Only this method can be used to call an implementation through an interface    
    }    
}  
```


## Lambda Expressions
Lambda expressions or anonymous functions are used in almost every programming language these days.  
Essentially, a lambda expression can be defined like this:  

An anonymous method that:  
* Has no access modifier (public, private etc.)
* Has no name
* Has no return statement

Lambda functions are used for convenience and readability. Below is a comparison between a normal function and a lambda function:  

```cs
static void Main(string[] args) {
    int number = 5;

    // Lambda functions using delegates (format: args => expression)
    // No argument lambda function: () => ...
    // One argument lambda function: x => ...
    // Multiple argument lambda function: (x, y) => ...
    Func<int, int> square = n => number*number;
    Console.WriteLine(square(number));

    // Normal function
    number = 5;
    Console.WriteLine(Square(number));
}

static int Square(int number) {
    return number*number;
}
```

Below is a lambda expression code example for filtering books. This kind of filtering by using a Predicate delegate as argument is extremely common:    

```cs
class Book {
    public string Title { get; set; }
    public int Price { get; set; }
}

class BookRepository {
    public List<Book> GetBooks() {
        return new List<Book> {
            new Book() {Title = "Title 1", Price = 5},
            new Book() {Title = "Title 2", Price = 7},
            new Book() {Title = "Title 3", Price = 17}
        };
    }
}

class Program {
    static void Main(string[] args) {
        var books = new BookRepository().GetBooks();

        var cheapBooks = books.FindAll(b => b.Price < 10);

        foreach(var book in cheapBooks) {
            Console.WriteLine(book);
        }
    }
}
```

## Events
A definition of events:  
* A mechanism for communication between objects. This means that when something happens inside an object it can notify other objects about it.
* Used in building *Loosely Coupled Applications*. This means that the classes or components are not tightly coupled together. A loosely coupled application is easy to extend without breaking or changing one or more of its existing capabilities.
* Helps extending applications.

If we look at the code example below, we add an extra line in order to also allow the Encode method to send a text message after encoding is complete. However, this means that VideoEncoder and all classes that depend on it have to be recompliled which is a bad thing. Events can help is solve this problem.

```cs
public class VideoEncoder {
    public void Encode(Video video) {
        // Encoding logic...

        _mailService.Send(new Mail());

        // If we add the following line to also send a text message after encoding, the whole class/method and all classes that depend on the VideoEncoder have to be recompiled.
        _messageService.Send(new Text());
    }
}
```

Events are basically a way to implement the Publish-Subscribe or Pub-Sub pattern where components or classes subscribe to a publisher. In our example above, the VideoEncoder could be the publisher/event sender while the MailService would be the subscriber/event receiver. VideoEncoder does not need to know anything about the MailService in order for this to work.

Below is a very simple code example illustrating the use of Events:

```cs
public class VideoEncoder {
    public void Encode(Video video) {
        // Encoding logic...

        OnVideoEncoded(); // Notify the subscribers that encoding has occurred.
    }
}

public class MailService {
    // A method implemented by the subscriber, an "Event Handler"
    public void OnVideoEncoded(object source, EventArgs e) {
         
    }
}
```

> Note: The Subscriber implements the *Event Handler* in the example above. The signature of this method is decided by a delegate in the Publisher.

**Delegate definition for Pub-Sub**  
* Agreement/contract between Publisher and Subscriber.
* Determines the signature of the event handler method in the Subscriber.

**Steps to follow in order to setup Publish-Subscribe**  
1. Define a delegate (the contract between the Publisher and Subscriber)
2. Define an event based on the delegate
3. Raise or publish the event

Below is a code example of a basic setup (note that there are almost duplicates of some lines in order to show how to pass data about the event that happened):  

```cs
public class Video {
    public string Title { get; set; }
}

public class VideoEventArgs : EventArgs {
    public Video Video { get; set; }
}

public class VideoEncoder {
    // The first parameter is always an object in C#.
    // EventArgs is typically any additional data that we want to send with the event.
    // The naming convention of the function is typically the name of the event followed by "EventHandler".
    public delegate void VideoEncodedEventHandler(object source, EventArgs args); // Without arguments

    public delegate void VideoEncodedEventHandler(object source, VideoEventArgs args); // With arguments

    public event VideoEncodedEventHandler VideoEncoded;

    public void Encode(Video video) {
        Console.WriteLine("Encoding video...");
        Thread.Sleep(3000); // Simulate video encoding

        OnVideoEncoded(video);
    }

    // Event Publisher methods should be protected, virtual and return void.
    // The naming convention is the word "On" followed by the name of the event.
    protected virtual void OnVideoEncoded(Video video) {
        // If there are any subscribers check
        if (VideoEncoded != null) {
            // The current class, "this" is publishing the event.
            // We pass EventArgs.Empty to not pass any data along.
            VideoEncoded(this, EventArgs.Empty); // Without arguments

            VideoEncoded(this, new VideoEventArgs() { Video = video }); // With arguments
        }
    }
}

class Program {
    static void Main(string[] args) {
        var video = new Video() { Title = "Video 1" };
        var videoEncoder = new VideoEncoder(); // Publisher
        var mailService = new MailService(); // Subscriber
        var messageService = new MessageService(); // Subscriber

        videoEncoder.VideoEncoded += mailService.OnVideoEncoded; // Setup event handler
        videoEncoder.VideoEncoded += messageService.OnVideoEncoded; // Setup another event handler

        videoEncoder.Encode(video);
    }
}

public class MailService {
    // Without arguments
    public void OnVideoEncoded(object source, EventArgs e) {
        Console.WriteLine("MailService: Sending an email...");
    }

    // With arguments
    public void OnVideoEncoded(object source, VideoEventArgs e) {
        Console.WriteLine("MailService: Sending an email..." + e.Video.Title);
    }
}

public class MessageService {
    // Without arguments
    public void OnVideoEncoded(object source, EventArgs e) {
        Console.WriteLine("MessageService: Sending a text message...");
    }

    // With arguments
    public void OnVideoEncoded(object source, VideoEventArgs e) {
        Console.WriteLine("MessageService: Sending a text message..." + e.Video.Title);
    }
}
```

There is an even simpler way to do this with predefined .NET delegate types. There are two different types:  
* EventHandler (without arguments)
* EventHandler<TEventArgs> (with arguments)

These types are basically always used instead of defining your own delegate when using events.

> Replace `public event VideoEncodedEventHandler VideoEncoded;` with `public event EventHandler<VideoEventArgs> VideoEncoded` and remove the delegate declaration in order to use the C# predefined EventHandler delegate.

## Extension Methods
The definition of extension methods:  
They allow us to add methods to an existing class without
* changing its source code
* creating a new class that inherits from it

Below is a code example:  

```cs
class Program {
    static void Main(string[] args) {
        string post = "This is supposed to be a very long post blah blah blah...";
        var shortenedPost = post.Shorten(5);
    }
}

namespace System {
    // A static class  is necessary to add extension methods to an existing class
    // The naming convention of the class you are extending followed by "Extensions" is a good practice
    public static class StringExtensions {
        // First parameter will always have the "this" keyword followed by the type you are extending
        public static string Shorten(this String str, int numberOfWords) {
            if (numberOfWords < 0)
                throw new ArgumentOutOfRangeException("numberOfWords should be greater or equal to 0.");
            if (numberOfWords == 0) return "";

            var words = str.Split(' ');

            if (words.Length <= numberOfWords) return str;

            return string.Join(" ", words.Take(numberOfWords)) + "...";
        }
    }
}
```

> In the example above we extend the String class with a Shorten Method. Note that the namespace of our StringExtensions class is System, thus whenever we import System in order to access String we also get our extension methods.

Only create extension methods when you have to. Usually you will use extension methods rather than create them yourself. An example of this is the IEnumerable interface. This interface has lots of extension methods that were added with the introduction of LINQ. See the next chapter for an in depth look at LINQ. 



## LINQ

## Nullable Types

## Dynamic

## Exception Handling

## Asynchronous Programming with Async/Await

## Polymorphism
Polymorphism is one one of the main aspect of OOPS Principles which include method overriding and method overloading. Virtual and Override keyword are used for method overriding and new keyword is used for method hiding.

* The virtual keyword is used to modify a method, property, indexer, or event declared in the base class and allow it to be overridden in the derived class.
* The override keyword is used to extend or modify a virtual/abstract method, property, indexer, or event of base class into derived class.
* The new keyword is used to hide a method, property, indexer, or event of base class into derived class.

Consider the below class hierarchy with classes A, B, and C. A is the super/base class, B is derived from class A and C is derived from class B.

A (super/base class) --> B --> C

### Method hiding
For hiding the base class method from derived class simply declare the derived class method with a new keyword.

```cs
using System;
namespace Polymorphism
{
    class A
    {
        public void Test() { Console.WriteLine("A::Test()"); }
    }

    class B : A
    {
        public new void Test() { Console.WriteLine("B::Test()"); }
    }

    class C : B
    {
        public new void Test() { Console.WriteLine("C::Test()"); }
    }

    class Program
    {
        static void Main(string[] args)
        {
            A a = new A();
            B b = new B();
            C c = new C();

            a.Test(); // output --> "A::Test()"
            b.Test(); // output --> "B::Test()"
            c.Test(); // output --> "C::Test()"

            a = new B();
            a.Test(); // output --> "A::Test()"

            b = new C();
            b.Test(); // output --> "B::Test()"

            Console.ReadKey();
        }
    }
}
```

### Method overriding
In C#, for overriding the base class method in a derived class, you have to declare a base class method as virtual and derived class method asoverride shown below:

```cs
using System;
namespace Polymorphism
{
    class A
    {
        public virtual void Test() { Console.WriteLine("A::Test()"); }
    }

    class B : A
    {
        public override void Test() { Console.WriteLine("B::Test()"); }
    }

    class C : B
    {
        public override void Test() { Console.WriteLine("C::Test()"); }
    }

    class Program
    {
        static void Main(string[] args)
        {
            A a = new A();
            B b = new B();
            C c = new C();
            a.Test(); // output --> "A::Test()"
            b.Test(); // output --> "B::Test()"
            c.Test(); // output --> "C::Test()"

            a = new B();
            a.Test(); // output --> "B::Test()"

            b = new C();
            b.Test(); // output --> "C::Test()"

            Console.ReadKey();
        }
    }
}
```

### Method overloading
You can also mix the method hiding and method overriding by using virtual and new keyword since the method of a derived class can be virtual and new at the same time. This is required when you want to further override the derived class method into next level as I am overriding Class B, Test() method in Class C as shown below:

```cs
using System;
namespace Polymorphism
{
    class A
    {
        public void Test() { Console.WriteLine("A::Test()"); }
    }

    class B : A
    {
        public new virtual void Test() { Console.WriteLine("B::Test()"); }
    }

    class C : B
    {
        public override void Test() { Console.WriteLine("C::Test()"); }
    }

    class Program
    {
        static void Main(string[] args)
        {
            A a = new A();
            B b = new B();
            C c = new C();

            a.Test(); // output --> "A::Test()"
            b.Test(); // output --> "B::Test()"
            c.Test(); // output --> "C::Test()"

            a = new B();
            a.Test(); // output --> "A::Test()"

            b = new C();
            b.Test(); // output --> "C::Test()"

            Console.ReadKey();
        }
    }
}
```

## Access Modifiers

* **public**: The type or member can be accessed by any other code in the same assembly or another assembly that references it.  
* **private**: The type or member can be accessed only by code in the same class or struct.  
* **protected**: The type or member can be accessed only by code in the same class, or in a class that is derived from that class.  
* **internal**: The type or member can be accessed by any code in the same assembly, but not from another assembly.  
* **protected internal**: The type or member can be accessed by any code in the assembly in which it's declared, or from within a derived class in another assembly.  
* **private protected**: The type or member can be accessed only within its declaring assembly, by code in the same class or in a type that is derived from that class.  

## yield return
The yield keyword signals to the compiler that the method in which it appears is an iterator block.  The compiler generates a class to implement the behavior that is expressed in the iterator block. In the iterator block, the yield keyword is used together with the return keyword to provide a value to the enumerator object. This is the value that is returned, for example, in each loop of a foreach statement. The yield keyword is also used with break to signal the end of iteration.

> yield creates a state machine "under the covers" that remembers where you were on each additional cycle of the function and picks up from there.

```cs
// BEFORE
public static IEnumerable func(int start, int number) {
    int[] _number = new int[number];
    for (int i = 0; i < number; i++) {
        _number[i] = start + 2 * i
    }

    return _number;
}

// AFTER
public static IEnumerable func(int start, int number) {
    for (int i = 0; i < number; i++) {
        yield return start + 2 * i
    }
}

```

## The out keyword


## Value types vs Reference types
In C#, these two data types are categorized based on how they store their value in the memory.

### Value type
A data type is a value type if it holds a data value within its own memory space. It means the variables of these data types directly contain values.

The following data types are all of value type:

bool  
byte  
char  
decimal  
double  
enum  
float  
int  
long  
sbyte  
short  
struct  
uint  
ulong  
ushort  

> When you pass a value-type variable from one method to another, the system creates a separate copy of a variable in another method. If value got changed in the one method, it wouldn't affect the variable in another method.

### Reference type
Unlike value types, a reference type doesn't store its value directly. Instead, it stores the address where the value is being stored. In other words, a reference type contains a pointer to another memory location that holds the data.

The followings are reference type data types:

String
Arrays (even if their elements are value types)
Class
Delegate

```cs
static void ChangeReferenceType(Student std2)
{
    std2.StudentName = "Steve";
}

static void Main(string[] args)
{
    Student std1 = new Student();
    std1.StudentName = "Bill";
    
    ChangeReferenceType(std1);

    Console.WriteLine(std1.StudentName);
}
```

> Output: Steve


