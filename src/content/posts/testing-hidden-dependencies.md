---

title: "Testing Legacy Code: Hidden Dependencies"
date: 2016-09-23T07:52:00+02:00
description: 'Let’s talk about The God Object, The Blob, that giant method with hundreds of lines that does a million things. You know it, it’s that class that powers the entire application and everyone in the team is afraid to change even a single line of code. '

---

Let’s talk about [The God Object](https://en.wikipedia.org/wiki/God_object), [The Blob](https://sourcemaking.com/antipatterns/the-blob), that giant method with hundreds of lines that does a million things. You know it, it’s that class that powers the entire application and everyone in the team is afraid to change even a single line of code.

{{< figure src="/images/testing-hidden-dependencies/1.jpg" >}}

Usually, that’s the class that “just works” because it has been polished with the tears of the poor developers who have fixed countless bugs during the long years of its existence. Any change requires an entire sprint just for impact analysis, and maybe a couple others for bug fixing and stabilization. And, unfortunately, new features often require changes in this class, just because it contains the most parts of the application.

At some point, someone decides enough is enough and something needs to be done. The safe thing to do is to write some tests for it, as a safety harness against breaking changes. And you are the brave soul who is gonna do it.

---

First, you will realize it’s impossible.

The code is so tangled that even if you manage to design a test for it, you can’t run it because it has calls to half of the application’s components, or it executes network requests, or it reads from a configuration file etc.

Let’s look at an example:

{{< gist corneliudascalu 403361073244bf8fd8a2cb3998dd370a >}}

This is only sample code. The real code would be orders of magnitudes larger, and peppered with null checks, try-catch blocks or log messages. And if you are not already familiar with it, it will be extremely difficult to grasp all its intricacies (which is a must, if you want to write useful tests).

# 1. Instantiating

Even just creating a new *PizzaHelper* object will most likely fail. Depending on the implementation of the *DbHelper*, it may access some APIs that are not working in a test environment.

In a real life scenario, the constructor of such a class will contain much more of these deep initializations.

If you actually think about it, the constructor is doing all this stuff just because the class **needs** those dependencies. It could just as well **receive** the dependencies, already created, from elsewhere.

```java
public PizzaHelper(PizzaDatabase db) {  
    this.pizzaDatabase = db;  
}
```

Now you can instantiate a *PizzaHelper* much more easily. For starters, you can pass a *null* as a parameter and see what happens. (This is a trick recommended by [Michael Feathers](http://www.goodreads.com/book/show/44919.Working_Effectively_with_Legacy_Code), as a way to quickly find out if the parameter is used and what is its potential impact). After that, you will probably pass in a test double, for example a *FakePizzaDatabase* which keeps data only in memory.

# 2. More dependencies

If you call the *buyPizza()* method now, you will also run into trouble. The *UserManager* will be created and maybe it will check for a logged in user. The *HttpClient* will want to send a request to the server. Both are too much for a unit test.

The quickest way to solve this would be to extract these dependencies as member variables and set them using constructor parameters or setter methods. This way you can pass in mock objects and actually run the test.

{{< gist corneliudascalu bf8bf51e5557832734abbfbdf86fee6c >}}

However, the test will be pretty complicated. You need to check if a call to *buyPizza()* results in a call to the mock *HttpClient* with a request string that matches the expected order.

# 3. Cleaning up

Once you have set up a test harness, the fun of breaking the class apart can start. Our sample class is not large, but it can already be cleaned up a lot. A real class has a lot more garbage in it.

The approach I like to use resembles a game of Jenga. You have to identify the easiest piece you can pull from the pile, and then the next one, and so on. If you follow strict TDD, you should first modify the unit tests, and then the actual code.

For example, one of the first things that are obvious is that the *PizzaHelper* doesn’t actually need a reference to the *UserManager*, but only the user id. So you can pass the user id as a parameter to the *buyPizza()* method, getting rid of one constructor parameter. (You also improve the code in ways not immediately obvious in this sample code. For example, in real code you would need to handle the case when the user is not logged in.)

The next thing to notice is the order being created inside the *buyPizza()* method. This breaks the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle). The order could be created by another class, an *OrderGenerator*, or an *OrderFactory*. This will allow you to group in the same class the logic for creating different kinds of orders.

One other thing to be done would be to pass the *Callback* as a method parameter. This will separate another responsibility outside the *PizzaHelper* class.

{{< gist corneliudascalu d9dc1921508046262e5afb83a93e2958>}}

Now the code looks a lot cleaner, but it could be cleaned up even further. For example, the *OrderFactory* could be passed as a constructor dependency, allowing you to mock it, or to modify the order creation logic in the future without touching the *PizzaHelper*. The four parameters of the *buyPizza()* method could be encapsulated in a small class, to improve the clarity of the code (as it is now, someone could call the method with the *count* and *userId* swapped and the compiler will happily go along with it).

# 4. Reality

In real code, it’s a lot more difficult to identify and apply these improvements. It’s important to understand that you cannot do it all at once. Most of the time, one small insignificant change helps you see another one, moving some code to a private method gives birth to a separate class, and step by step your class starts looking leaner. It’s a continuous process, which requires discipline. You have to stop from time to time to clean up the accumulated dirt before it starts rotting.

> Perfection is achieved, not when there is nothing more to add, but when there is nothing left to take away — [Antoine de Saint-Exupéry](https://www.goodreads.com/quotes/19905)