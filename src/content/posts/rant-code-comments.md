---
title: "Where I Rant Against Code Comments"
date: 2016-12-05T08:11:36+02:00
description: 'I hate code comments. I hate reading comments and I hate writing comments. It’s one of the first things I notice in a new project, and it’s usually a red flag about the code quality.'
image: images/rant-code-comments/1.jpg
---

I hate code comments. I hate reading comments and I hate writing comments. It’s one of the first things I notice in a new project, and it’s usually a red flag about the code quality.

I know people that swear by code comments. They feel helpless if the code is not commented extensively. And they usually have several arguments for why comments are needed.

> I have to explain what a method does.

```java
/**
 * Retrieves the registration info of the user
 * @param args The user's id
 */
public String getInfo(int args){
  ...
}
```

I agree that `String getInfo(int args)` is not a method you would like to use without a comment explaining what it actually does. But why don’t you use a better name for it? It doesn’t cost you anything to rename it:

```java
/**
 * Retrieves the registration info of the user
 * @param userId The user's id
 */
public String getRegistrationInfo(int userId){
  ...
}
```

Wow! It looks like the comment is not needed anymore, is it? It’s actually duplicating information and violating the [DRY principle](https://en.wikipedia.org/wiki/Don't_repeat_yourself).

Sadly, this kind of redundant comments is much too often found in actual code. Many times the comments are added because of stupid coding style rules mandating every method to be commented. And what’s a poor programmer to do? Just add a comment to satisfy the law…

```java
/**
 * Set the user's name
 */
public void setName(String name){
  ...
}
```

If I received a dollar each time I deleted such a comment, I could probably make a living out of just deleting useless comments.

> But it’s not doing any harm!

Except for making code harder to read, yes, it’s not doing any harm. For now. But I guarantee you that at one point in the future somebody will need separate methods to get/set the first and last name. They will most likely use their IDE’s refactoring tool and rename the method, forgetting to rename the comment as well:

```java
/**
 * Set the user's name
 */
public void setLastName(String lastName){
  ...
}
```

BAM! Misleading comments. Extrapolate to several years of this and you start to slowly go insane.

> But what if I can’t find a good name for a method?

```java
/**
 * Display an error message to the user, log the error
 * and navigate to the login screen
 * 
 * @param exception The exception
 */
public void handleError(Exception exception){
  ...
}
```

I agree, [naming things is difficult](http://martinfowler.com/bliki/TwoHardThings.html). Start by actually writing the best name you can think of. If `displayAndLogErrorThenGoToLogin()` doesn’t exactly roll off the tongue, ask yourself why. Ah, too many verbs, you say? Maybe it’s because the method does too many things? May I remind you of a little thing called the [Single Responsibility](https://en.wikipedia.org/wiki/Single_responsibility_principle) principle? Why don’t you split it into 3 smaller methods: `displayErrorMessage(Exception e)`, `writeToLog(Exception e)` and `navigateToLoginScreen()`?

> I need to document this method with ambiguous arguments

```java
/**
 * Display a dialog at a certain position on the screen
 * 
 * @param type One of Dialog.ALERT, Dialog.ASK or Dialog.NEGATIVE
 * @param isBlocking True if the dialog should block the UI
 */
public void showDialog(int x, int y, int type, boolean isBlocking){
  ...
}
```

Oh boy… Where do I begin?

Boolean arguments? 99% of the time, that’s a method begging to be split in two. How about `showBlockingDialog()` and `showNonBlockingDialog()`?

You have to document the supported values for a parameter? What’s stopping someone from passing in [42](https://www.google.ro/search?q=what%27s+the+answer+to+life+the+universe+and+everything)? Or `Integer.MIN_VALUE`? Create an enum to hold the possible values and restrict anyone from passing in a wrong value. Or consider using a factory to create different types of dialogs.

> But you see, I have this one weird trick I need to document…

```java
...
Blob blob = getLastUsedBlob();
// the blobs are spread out in memory to make attacks difficult
int nextAddress = blob.getMemoryAddress() >> blob.getByteSize();
Blob nextBlob = Blob.getFromAddress(nextAddress);
```

I agree, this is the kind of obscure knowledge that needs to be documented. Sometimes it helps to encapsulate this knowledge in a separate method or class. In this specific example, it would make sense to encapsulate it in the `Blob` class itself:

```java
/**
 * Find the next blob
 */
public Blob getNextBlob(){
  // the blobs are spread out in memory to make attacks difficult
  int nextAddress = getMemoryAddress() >> getByteSize();
  return Blob.getFromAddress(nextAddress);
}
```

Now, not only you made life easier for anyone who works with blobs, you also made your code safer. No more accidental `<<` instead of `>>`. Yay!

----

Code can be such a beautiful thing. And, like most beautiful things, it takes hard work to create. Unfortunately, it’s easy to write bad code. And [bad code](https://books.google.ro/books?id=5wBQEp6ruIAC&lpg=PA29&ots=n5npfteIqX&dq=the%20pragmatic%20programmer%20bad%20code%20requires%20comments&hl=ro&pg=PA29#v=onepage&q=the%20pragmatic%20programmer%20bad%20code%20requires%20comments&f=false) [*requires*](https://books.google.ro/books?id=5wBQEp6ruIAC&lpg=PA29&ots=n5npfteIqX&dq=the%20pragmatic%20programmer%20bad%20code%20requires%20comments&hl=ro&pg=PA29#v=onepage&q=the%20pragmatic%20programmer%20bad%20code%20requires%20comments&f=false) [comments](https://books.google.ro/books?id=5wBQEp6ruIAC&lpg=PA29&ots=n5npfteIqX&dq=the%20pragmatic%20programmer%20bad%20code%20requires%20comments&hl=ro&pg=PA29#v=onepage&q=the%20pragmatic%20programmer%20bad%20code%20requires%20comments&f=false). But don’t delete the comments and call your code beautiful. Write it in a way that doesn’t need comments in the first place.

So go forth and code (without comments)!