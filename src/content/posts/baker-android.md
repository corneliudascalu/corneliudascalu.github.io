---

title: "Baker Journey Android App"
date: 2022-01-16T08:27:20+02:00
description: 'I start my series on side-by-side app development with a basic Android implementation of Baker Journey. Nothing fancy, just plain old Android SDK.'
image: 
draft: true

---

# Baker Journey Android App

Starts with configuring the latest and greatest Gradle plugin, not without any hurdles. For some reason, Android Studio decides to select JDK 15 as default and I waste an hour trying to figure out why the Gradly Sync step fails with a very helpful message `Caused by: org.codehaus.groovy.control.MultipleCompilationErrorsException: startup failed`

Next, update the dependencies to the latest versions. Why is this even necessary? I just started a new project from scratch, it defaults to older library versions and warns me to upgrade. Couldn't this be done automatically?

Cheated and copied a lot of the code I previously wrote for a sample app.

I have to constantly fight the temptation to go down the rabbit hole: theming, testing, architecture, even release publishing.



- app bundle required

- generate upload key

- set up gradle.properties in user home with alias and password