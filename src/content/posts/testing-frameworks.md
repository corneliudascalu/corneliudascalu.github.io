---

title: "Testing Legacy Code: Frameworks"
date: 2016-10-13T08:01:59+02:00
description: 'One often encountered feature of legacy code is a tight coupling to an external framework or SDK.'

---

One often encountered feature of legacy code is a tight coupling to an external framework or SDK. It may be in the form of a “Manager” or “ApiClient” singleton, used throughout the application in the most critical areas. Or it may be more subtle, like making use of a “DateUtil” class provided by the framework.

You may ask “What’s your problem? Why should I reinvent the wheel?”. No, you shouldn’t. Hear me out.

Using libraries, SDKs, frameworks etc. is a good thing, especially when they are mature and battle-tested. [There is no point in rolling your own HTTP client or date util](https://www.youtube.com/watch?v=-5wpm-gesOY). The problem is when they are used without thinking how you would replace them.

Because sometimes libraries are deprecated. Or new versions contain breaking changes. Or a newer and fancier library is released, and you are stuck with code written when the dinosaurs were still a thing.

Or you just want to write some unit tests. And when you have legacy code peppered with framework method calls, that’s when the fun starts.

## Singletons

Oh, yes, everybody loves to [hate on singletons](https://corneliudascalu.com/posts/testing-legacy-code). It’s even worse when the code uses singletons from an external library.

```java
public void loadImage(){  
  Picasso.with(context).load(url).into(view);  
}
```

This is a beautiful line of code. Any Android developer will tell you that. The day Jake Wharton published [Picasso](https://github.com/square/picasso) should be year zero in the Android community.

But try to write a unit test for that method. Or even better, imagine how real code looks, for example a well aged list adapter, bearing the scars of countless bug fixes and cleanups.

You could modify the code to supply a reference to Picasso through dependency injection, and mock it in the unit tests. It will work pretty well.

But imagine you have to switch to another library, like [Volley](https://developer.android.com/training/volley/index.html) (*why would anyone use anything else than Picasso? Anyway…*). You would need to change your code to something like this:

```java
public void loadImage(){  
  imageLoader.get(url, ImageLoader.getImageListener(view,   
    R.drawable.default, R.drawable.error));  
}
```

And of course, your tests will have to be rewritten as well.

What if you used a thin wrapper around Picasso, like this:

```java
public interface GenericImageLoader {
  void loadImage(String url, ImageView imageView);
}
public class PicassoImageLoader implements GenericImageLoader {
  public void loadImage(url, view){
    Picasso.with(context).load(url).into(view);
  }
}
/// usage in the code: 
private GenericImageLoader loader = new PicassoImageLoader();
public void initializeStuff(){
  this.loader.loadImage("http://...", profileImageView);
}
```

You would have to accept a compromise and not unit test the wrapper. If you use it only as a wrapper, the method calls will map directly to library methods. As long as you don’t do anything wild in there, you should be safe.

Now, unit testing the *initializeStuff()* method is more straightforward. And if you had to switch to Volley, well, you would just create a different implementation of the *GenericImageLoader* interface:

```java
public class VolleyImageLoader implements GenericImageLoader {
  public void loadImage(url, view){
    imageLoader.get(url, ImageLoader.getImageListener(view,
      R.drawable.default, R.drawable.error));
  }
}
// usage in the code:
private GenericImageLoader loader = new VolleyImageLoader();
public void initializeStuff(){
  this.loader.loadImage("http://...", profileImageView);
}
```

The *initializeStuff()* method remains unchanged, but there’s a [whole new world](https://www.youtube.com/watch?v=t9-CS2v8wcc) (*ahem!)* behind the *GenericImageLoader* interface. Yay polymorphism!

## Static methods

Now, imagine you have a utility class with static methods in a method you want to test. Normally, if the static methods don’t do anything magic, they are not a problem. But then you have and API designed by someone that *reeealy* loves static methods. Like this way of [requesting location updates](https://developer.android.com/training/location/receive-location-updates.html#updates) in Android:

```java
public void startLocationUpdates() {
    LocationServices.FusedLocationApi.requestLocationUpdates(
            mGoogleApiClient, mLocationRequest, this);
}
```

Good luck testing that. You cannot even mock it. Your only chance is to create a wrapper around the API:

```java
public interface LocationApi {
  void requestLocationUpdates(GoogleApiClient client,                       
              LocationRequest request, LocationListener listener);
}
public class FusedLocationApiWrapper implements LocationApi {
  public void requestLocationUpdates(GoogleApiClient client,                       
              LocationRequest request, LocationListener listener) {
    LocationServices.FusedLocationApi.requestLocationUpdates(     
                    client, request, listener);
  }
}
// usage in code 
private LocationApi wrapper = new FusedLocationApiWrapper();
public void startLocationUpdates() {
    wrapper.requestLocationUpdates(
            mGoogleApiClient, mLocationRequest, this);
}
```

Again, as long as you’re only wrapping the static methods, you can live without tests.

However, you will probably realize that the *LocationApi* interface is still tightly coupled to the *FusedLocationApi.* It uses the *LocationRequest* class, needs a *GoogleApiClient* and a *LocationListener,* all part of Google’s Play Services.

To create a truly abstract layer over the location API, you have to define your own way of supplying the information contained in the *LocationRequest*.

```java
public interface LocationApi {
  void requestQuickLocation();
  void requestPreciseLocation();
}
public class FusedLocationApiWrapper implements LocationApi {
  private OnLocationListener listener;
  public FusedLocationApiWrapper(GoogleApiClient client){
    this.client = client;
  }
  public void requestQuickLocation(GoogleApiClient client,                       
              LocationRequest request, LocationListener listener) {
    LocationRequest request = LocationRequest.create();
    request.setMaxWaitTime(ONE_SECOND);
    request.setPriority(PRIORITY_BALANCED_POWER_ACCURACY);
    LocationServices.FusedLocationApi.requestLocationUpdates(     
                    client, request, listener);
  }
  public void requestPreciseLocation(GoogleApiClient client,                       
              LocationRequest request, LocationListener listener) {
    LocationRequest request = LocationRequest.create();
    request.setMaxWaitTime(FIVE_SECONDS);
    request.setPriority(PRIORITY_HIGH_ACCURACY);
    LocationServices.FusedLocationApi.requestLocationUpdates(     
                    client, request, listener);
  }
}
// usage in code
private GoogleApiClient client;
private LocationApi wrapper = new FusedLocationApiWrapper(client);
public void startLocationUpdates() {
    wrapper.requestQuickLocation();
}
```

The client code looks a lot cleaner, is easier to unit test and it’s easier to swap in another location provider. And you get to write a lot of extra code! What’s not to like? (*Don’t answer that*).

## Final classes and methods

Libraries and frameworks sometimes mark their public classes and methods as final. This makes a lot of sense from a security point of view. As an API creator, you don’t want developers to have too much fun with the internals of your API. So you try to keep your public facing interface as airtight as possible: final methods, singletons to enforce a single instance etc.

But you cannot mock them. See the above.

## Conclusion

[Encapsulation](https://en.wikipedia.org/wiki/Encapsulation_(computer_programming)) is important. [Separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns) is important. [Low coupling](https://en.wikipedia.org/wiki/Coupling_(computer_programming)) is important. [Dependency inversion](https://en.wikipedia.org/wiki/Dependency_inversion_principle) is important.

But it’s more important to know they are important. You don’t always have the time to abstract away everything. And neither you should. Just remember that external libraries are powerful beasts. It’s best to keep them at arm’s length.