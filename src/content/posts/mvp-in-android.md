---
title: "Model-View-Presenter in Android"
date: 2015-11-24T19:30:21+02:00
description: 'Experiment with the MVP architecture paradigm using Dagger for dependency injection'
// image: images/cctv.jpeg

---

## Intro

A while ago I stumbled upon this article by Antonio Leiva about implementing the MVP architecture in Android. I’ve always wanted to organize my code better, and MVP sounded like a good idea.
I theory, Android is all about MVC, with the Activity or Fragment being the controller and the view defined in XML layouts. In practice, you end up with a lot of view logic in the Activity. The MVP approach promises to solve this by assuming that the Activity or Fragment is the view, controlled by a Presenter.
I’m only going to comment on the article and add my experiences, so you should first read the article I mentioned above and the followup articles on dependency injection using Dagger.
Alright! First off, the sample code is too simple, and there are probably some places where you wondered: “What if I wanted to do this or that?”. I know I did. So I wanted to do something a little closer to a real app, and also integrate a testing framework (Espresso).
My sample app is a very simple note-taking app, MVPNotes. The app allows the user to write and save a note, display the existing notes in a list, delete a note or view note details. The code can be found on Github, if you want to check it out straight away.

## Setup

Before diving into the actual implementation, let me tell you a few things about setting up the project. I’m using Android Studio and Gradle. If you are using Ant or another build system, you should be able to translate the instructions, because I’m not using anything too fancy.
So, here is the build.gradle file. (BTW, I recommend Octotree, a very useful extension for navigating Github repos, available for Chrome and Firefox). The first important thing in the build script is the testInstrumentationRunner bit. That sets Espresso for testing. Note that I’m actually using Double Espresso, which is Jake Wharton’s port of Espresso to Gradle. You can see it included in the list of dependencies at androidTestCompile. You should also notice the included Dagger dependencies. To avoid a conflict between the Dagger dependency and the already included Dagger in Double Espresso, I’m excluding it from the test configuration:

```
exclude group: 'com.squareup.dagger'
```

Depending on your code, you may encounter other conflicts, listed by Jake Wharton in the readme file of Double Espresso.
One more note, Espresso fails on Android L Preview, as noted here.

### Setting up Dagger

I’ve used a slightly tweaked way of setting up the Object Graph than what you saw on Antonio Leiva’s blog. You can check it on Fabio Collini’s Github repo, TestableAndroidApps. The ObjectGraphHolder is a class used to store and get a reference to the application object graph, but also to force another object graph (useful when testing).
However, I’ve kept the approach suggested by Antonio of having a base Activity with an abstract getModules() method, in order to create a scoped object graph.
OK, I think I’ve covered all the important stuff, let’s get to the actual code.

## Main Logic

First, let’s define the model.

```java
public class Note { 
    public long id; 
    public String title; 
    public String text; 
    public DateTime createdDate; 
}
```

Let’s define the view that will display the list of notes and let the user write a new note. I’m trying to keep the code size to a minimum, so I’m not defining a deleteAll() method, for example. The public-facing interface of the view will let another part of the code to add a list of notes, remove a certain note from the view, or signal to the view to display a certain error.

```java
public interface NotesView { 
    void addNotes (Note... note); 
    void removeNote (Note note); 
    void setNoteError (String error); 
}
```

That another part of the code will be the NotesPresenter. It exposes methods for someone to request the existing notes from the database, to submit a new note or delete a note from the database.

```java
public interface NotesPresenter { 
    void requestNotes(); 
    void submitNewNote (String title, String text); 
    void deleteNote (Note note); 
}
```

The NoteInteractor will handle the interaction between the presenter and the database.

```java
public interface NoteInteractor { 
    void storeNote (Note note, OnNoteOperationListener listener);
    void deleteNote (Note note, OnNoteOperationListener listener); 
    void getNote (long id, OnNoteOperationListener listener); 
    void getAllNotes (OnNoteOperationListener listener); 
}
```

## Implementation

### NotesActivity

Let’s discuss the code of the NotesActivity. I won’t go through it step by step, I’ll only mention the interesting parts.
So, first of all, notice the fact that the NotesPresenter is injected by Dagger. Of course, you probably saw that NotesActivity extends BaseInjectedActivity and overrides the getModules() method to add the NotesModule to the object graph.
Let’s take a look at the onCreate() method. Check out the ArrayAdapter used to display the notes in the notesList ListView. You may argue that at least the ArrayList of notes, if not the adapter too, belong in the model and not the view. You could probably do it that way too, but I consider the adapter and the corresponding list of notes to be strictly View properties. They are owned and touched only by the NotesView and no one else.
Next, take a look at the last line of the onCreate() method.
    notesPresenter.requestNotes();
Now that is controversial. The view is basically controlling the presenter. It’s something that is not covered by Antonio Leiva, and I couldn’t find a better solution. An alternative would be to have an equivalent onCreate() method in the presenter, called from the activity’s onCreate(). The presenter would know that it’s time to start loading notes from the database and set them on the view.
However, that would create a tighter coupling between view and presenter. If I wanted to implement NotesView Beyond simple implementation
in a Fragment, I would have to think twice about the fragment lifecycle. Depending on the actual code in the presenter, it may not work with a fragment. But what if I wanted to use a plain old View? It would definitely not work without changing the presenter’s code. And that is bad. We want to keep the view and presenter oblivious of their internal implementations, otherwise the whole point of using MVP is lost.
So, I think allowing the view to signal to the presenter that it has finished initializing and is ready to receive notes is a good compromise. We are forced to do this by Android, because the Activity is created before the presenter. If we used a Fragment or View, we could call requestNotes() from a parent Activity and keep the NotesPresenter isolated from the NotesView.
Anyway, we can look at the requestNotes() call as just another event, like submitNewNote() and deleteNote().
The rest of the activity code is self-explanatory. I’ll get into the NoteDetailsDialogFragment later.

### SimpleNotesPresenter

There’s not much to say about the presenter code. The interaction with the view and with the interactor is very simple.
Just one remark: for clarity, I’m using the OnNoteOperationListener to get the results from the NoteInteractor. In a real app, you should use something more sophisticated, like an event bus.

### NoteInteractorImpl

The interactor is quite simple. Besides actually comunicating with the database, it simulates longer operation times, and generates random errors. It gets injected with a SimpleDatabase instance from the DatabaseModule.

### SimpleDatabase

The database is far from a real database. It’s just storing key-value pairs in a SharedPreferences instance. As you can see, it doesn’t even have an update() method, so it doesn’t even have a full set of CRUD operations.

## Beyond simple implementation

If you got this far, you probably have a pretty good idea about MVP and Dagger. Let’s see what happens if we want to do something a little more complex. We’ll display some note details in a DialogFragment when the user selects a note.
Check the NoteDetailsView and NoteDetailsPresenter.

```java
public interface NoteDetailsPresenter { 
    void setData(Note note); void viewReady(); 
    void deleteNote(); 
} 
public interface NoteDetailsView { You will notice the setData(Note) and viewReady() methods in the presenter. The setData() method initializes the presenter with a Note. If we want to keep everything the Android way, the presenter will be created inside the dialog, so setData() will have to be called from the dialog (we could set the presenter on the dialog after it’s created and before it’s displayed, but that is quite fishy). It looks pretty bad, and I couldn’t find a better way to do it.
```

As an alternative, we could let the presenter itself get the note from the dialog. Something like:

```java
    void setNote(Note note); 
    void close(); 
    void showError(String error); 
}
```

You will notice the setData(Note) and viewReady() methods in the presenter. The setData() method initializes the presenter with a Note. If we want to keep everything the Android way, the presenter will be created inside the dialog, so setData() will have to be called from the dialog (we could set the presenter on the dialog after it’s created and before it’s displayed, but that is quite fishy). It looks pretty bad, and I couldn’t find a better way to do it.
As an alternative, we could let the presenter itself get the note from the dialog. Something like:

```java
NoteDetailsView 
--- 
onCreate() { 
    ... 
    Note note = getArguments().getParcelable("note_extra"); 
    presenter.dataReady(); 
    ... 
} 
NoteDetailsPresenter 
--- 
dataReady(){ 
    this.note = notesView.getNote(); 
}
```

This removes the posibility of setting the data from outside the view-presenter couple. Also, it’s a litle more brittle than my approach. Where before we could set the data at any time (theoretically), now the data is set only in onCreate(). It doesn’t make any difference in our case, but it could, in a more complex implementation.
Anyway, the point is that we have too much coupling between view and presenter. If anyone has a better solution, please let me know.
(Of course, I’m well aware that there’s no need to keep the note in the presenter, in our case. I’m just trying to think of what I should do if I really needed to pass some data from the dialog to the presenter and keep it there).
The rest of the logic is pretty simple. I should note that the code in onNoteDeleteError() could be better, but that’s beyond the scope of this app. You may notice that I’m using the same NoteInteractor to delete the note. Yay code reuse!

## Testing

The only thing I want to highlight for testing is the way I override the NoteInteractor implementation with MockNoteInteractor. First, I’m calling ObjectGraphHolder.forceObjectGraphCreator() in the setUp() method of the instrumentation test, setting a new object graph composed of the AppModule and MockNoteInteractorModule. Note that the AppModule already provides a NoteInteractor implementation, but the overrides = true annotation in the MockNoteInteractorModule takes care of that. Also, I could have used another AppModule if I wanted.

## Conclusion

My personal conclusion is that I need to experiment more. It’s still not clear to me how this approach would scale in a large application.
But would I use it? Yes. It’s better than jamming all the code in the Activity or Fragment.
