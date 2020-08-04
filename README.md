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
public interface ICustomerFilter 
{
    bool Filter (Customer customer); 
}

public class Util 
{
    public static List<Customer> FilterCustomers (List<Customer> allCustomers, ICustomerFilter customerFilter) 
    {
        List<Customer> filteredCustomers = new List<Customer>();
        for(int i = 0; i < allCustomers.Length; i++)
        {
            if(customerFilter(allCustomers[i])) filteredCustomers.add(allCustomers[i]);
        }
        return filteredCustomers;
    } 
}

class CustomersWithFirstAndLastNames: ICustomerFilter 
{
    public bool Filter (Customer customer) => String.IsNullOrWhiteSpace(customers?.firstName) && String.IsNullOrWhiteSpace(customers?.lastName) 
}

class CustomersWhoLikeStarks: ICustomerFilter
{
    public bool Filter (Customer customer) => customer?.gameOfThronesHouse == "Starks";
}
```

```cs
public delegate bool ICustomerFilter(Customer customer);

public class Util 
{
    public static List<Customer> FilterCustomers (List<Customer> allCustomers, ICustomerFilter customerFilter) 
    {
        List<Customer> filteredCustomers = new List<Customer>();
        for(int i=0; i<allCustomers.Length; i++)
        {
            if(customerFilter(allCustomers[i])) filteredCustomers.add(allCustomers[i]);
        }
        return filteredCustomers;
    } 
}

static bool CustomersWithFirstAndLastNames(Customer customers) => 
    String.IsNullOrWhiteSpace(customers?.firstName) &&
    String.IsNullOrWhiteSpace(customers?.lastName);

static bool CustomersWhoLikeStarks(Customer customers) => 
    customers?.gameOfThronesHouse == "Starks";
```

Comparing usage:

```cs
static void Main() 
{
    // Interface:
    List<Customer> customers = new List<Customer>();
    Util.PopulateCustomers(out customers);
    var myFilteredCustomers = Util.Filter(customers, new CustomersWithFirstAndLastNames());
    
    // Delegates:
    List<Customer> customers2 = new List<Customer>();
    Util.PopulateCustomers(out customers2);
    var myFilteredCustomers = Util.Filter(customers2, CustomersWithFirstAndLastNames);
}
```

The difference is subtle, but for this scenario, the delegate enables cleaner design. Technically, a delegate is a little bit overkill for this usecase as we could achieve the above with LINQ, but it does provide a clear comparison against interfaces.

Let’s say that now we needed to combine multiple filters so we would get customers who have CustomersWithFirstAndLastNames and CustomersWhoLikeHouseStark. Here is how we would implement it with an interface and a delegate:

```cs
static void Main() 
{
    // Interface:
    ...
    var myFilteredCustomers = Util.Filter(
          Util.Filter(customers, new CustomersWithFirstAndLastNames()),
               new CustomersWhoLikeHouseStark())
          );

    // Delegates:
    ...
    ICustomerFilter manyFilters = CustomersWithFirstAndLastNames;
    manyFilters += CustomersWhoLikeHouseStark;
    var myFilteredCustomers = Util.Filter(customers2, manyFilters);
}
```

For the interface example, it is obvious that we could potentially end up with a heavily nested filter, which is not ideal. We could have instead executed Util.Filter(...) on multiple lines and reassigned the result to the list, but this is not as succinct or flexible as the multicast delegate. Do note, in the delegate example, we could also remove the filter by applying -=.

## Lambda Expressions

## Events

## Extension Methods

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


