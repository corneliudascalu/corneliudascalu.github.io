---

title: "Testing Legacy Code: Singletons"
date: 2016-07-11T07:36:41+02:00
description: 'You know that guy that works on that messy codebase? The guy that always has a story about some weird bug he spent several days on?'

---

You know that guy that works on that messy codebase? The guy that always has a story about some weird bug he spent several days on? If you ask him what is the worst part about that code, his answer might be something like

{{< figure src="/images/testing-legacy-code/1.jpg" >}}

f you don’t know [why that is bad](http://stackoverflow.com/questions/137975/what-is-so-bad-about-singletons) consider yourself lucky. If you just chuckled bitterly, then you know what a plague they are when you need to put your code in a test harness.

Imagine the following piece of code:

```java
public void orderPizza(String type, int quantity){  
    Address address = UserManager.getInstance().getHomeAddress();  
    // do stuff   
    OrderManager.getInstance().placeOrder(type, quantity, address);  
}
```

A useful test might verify that calling the *orderPizza()* method results in an order with the correct type and quantity placed for the correct address. Without touching the method’s code, you will need at least to create a fake user somehow, and to verify that an order has been placed in the database. You will quickly realize that this path leads to madness: you will probably have to authenticate the fake user and maybe initialize the *OrderManager*.

Ideally, you want only to verify that the *placeOrder()* method was called with the correct parameters. Usually, this is achieved by using a [test double](http://www.martinfowler.com/bliki/TestDouble.html) for the *OrderManager*. But that’s impossible, because by definition, a singleton allows creating only a single instance of the class, and doesn’t let you change it. So, unless you change something in the code, you are stuck using the real *OrderManager*.

So let’s see what you could change in the code, shall we? Of course, you want to do small changes to avoid breaking the code before you even have a chance to add tests.

## Extract Parameter

You could extract the *OrderManager* and *UserManager* as method parameters. The method signature will be something like this:

```java
public void orderPizza(OrderManager orderManager, UserManager userManager, String type, int quantity)
```

It looks decidedly messier, but it lets you pass fake managers to the method. Of course, you will have to modify the code everywhere this method is called. It could mean a lot of changes, each one a possible source of bugs.

## Add a set method to the singleton

You could give up the singleton’s “single-ness” and allow setting a different instance through a static *setInstance()* method. This will let you set a fake instance before running the test. Obviously, this will let anyone else set a different instance anywhere in the code. And this is how you discover again why [global state is evil](http://programmers.stackexchange.com/questions/148108/why-is-global-state-so-evil).

You will have to add warning comments and annotations to the method to make it very clear that it’s only intended to be used in tests. Murphy’s law guarantees that they will be ignored sooner or later and all hell will break loose.

## Use a singleton [service locator](https://en.wikipedia.org/wiki/Service_locator_pattern)

This is my favorite method, because the changes are the less invasive. It involves creating a master singleton that contains methods to supply a reference to any singleton used in the code. For example, you will use this to get the *OrderManager* in the *orderPizza()* method:

```java
SingletonLocator.getInstance().getOrderManager();
```

To allow testing, you still need a *setInstance()* method on the *SingletonLocator*. The advantage is that only this singleton will have it, and it’s easier to make sure it’s not used outside test code.

Using this approach, the *orderPizza()* method will look like this:

```java
public void orderPizza(String type, int quantity){  
    Address address = SingletonLocator.getInstance().getUserManager().getHomeAddress();  
    // do stuff   
    SingletonLocator.getInstance().getOrderManager().placeOrder(type, quantity, address);  
}
```

And a test using the [Mockito](http://mockito.org/) framework might look like this:

```java
public void testOrderPizza(){ 
    // setup  
    SingletonLocator mockLocator = mock(SingletonLocator.class)  
    when(mockLocator.getUserManager())  
        .thenReturn(mockUserManager); 
    when(mockLocator.getOrderManager())  
        .thenReturn(mockOrderManager); 
    SingletonLocator.setInstance(mockLocator); 

    // execution  
    new FoodDelivery().orderPizza("Quattro Stagioni", 1); 

    // verification  
    verify(mockOrderManager).placeOrder("Quattro Stagioni", 1, mockUserManager.getHomeAddress());  
}
```

Of course, whenever you have the chance, you should refactor the code, extract the hidden dependencies, and leave the code cleaner than you found it. But we don’t always get the time to do that, and we have to choose our battles.

This is a quick way to make testable code out of a hot mess of singletons. It won’t make the code itself better, but it will let you surround it with tests, to give you that bit of confidence you need when you make small changes.