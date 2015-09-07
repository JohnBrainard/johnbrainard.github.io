---
layout: post
title: "Android Photoframe in Kotlin: Part 1 - Introduction"
excerpt: "In this post, I'll introduce how I built an Android photoframe app to learn Kotlin."
tags: [code, kotlin, java, android, photoframe]
modified: 2015-09-01
comments: true
---

As part of my exercise to learn [Kotlin](http://kotlinlang.org/), I wrote a simple photoframe in Android to show off images from my [photography site](http://brainardphotography.com). The app we build here will use a static set of images to start and possibly use the Flickr API to pull in the Explore images.

I'm going to make the assumption that you're familiar with Android already and know how to set up your environment. In this series, we'll be using Android Studio to create the project and 4.0.3 will be our target platform.

The posts in this series will roughly be:

* Part 1: Setting up the project with Gradle and Android Studio
* Part 2: [Introducing the main views]({% post_url 2015-09-02-android-photoframe-in-kotlin-part2 %})
* Part 3: [Creating the image source and animating the transitions]({% post_url 2015-09-07-android-photoframe-in-kotlin-part3 %})
* Part 4: Handling power connect/disconnect to start the app and enable/disable screen timeout
* Part 5 & beyond: If there are any refinements to the code we can make, we'll revisit this series

At the end of this series, I'll post a link to the Github repository for this project. Each commit will represent a step in this process which you can use to verify your results and troubleshoot if necessary. I'll be making sure that the code works and using that as the source for code examples in this series. It can be very frustrating to get through several steps and find out there was a mistake in the instructions and be unsure how to deal with them.

### Setting up

We'll begin by creating a new Android Studio project.

1. Click "Start a new Android Studio project" on the Welcome screen
2. Enter "Kotlin Photoframe" for Application Name
3. Enter "johnbrainard.github.io" for the Company Domain, or whatever you feel is appropriate. Note that this is a domain name, not a package name. Android Studio will reverse the order of the parts to make `io.github.johnbrainard.kotlinphotoframe`.
4. Accept the default project location or select a new one then click next.

In the *Target Android Devices* page:

1. Select *Phone and Tablet*,
2. Choose *API 15: Android 4.0.3 (Icecream Sandwich)* as the Minimum SDK then
3. Click *next* to continue

In the *Add an activity to Mobile* page,

1. Select *Blank Activity* and
2. Click *next* to continue

In the *Customize the Activity* page

1. Leave everything as is and click *Finish*

You should be able to run the app at this point.

### Setting up Android Studio to use Kotlin

There are two IntelliJ plugins you'll need to install, which can be done through Android Studio's preferences. They are:

1. Kotlin
2. Kotlin Extensions for Android

After you have the Kotlin plugins installed, you'll need to modify the `app/build.gradle`.

Add the following to your `app/build.gradle` script. This sets up Gradle to be able to use the *kotlin-android* plugin.

{% highlight groovy %}

buildscript {
	ext.kotlin_version = '0.12.1230'
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
		classpath "org.jetbrains.kotlin:kotlin-android-extensions:$kotlin_version"
	}
}

{% endhighlight %}

At the top of `app/build.gradle`, insert this:

{% highlight groovy %}

apply plugin: 'kotlin-android'

{% endhighlight %}

And in your `dependencies` configuration, add the following:

{% highlight groovy %}

compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
compile "org.jetbrains.kotlin:kotlin-android-extensions:$kotlin_version"

{% endhighlight %}

Once you've made these changes, Android Studio will ask you to resync your Gradle scripts. If not, you can force a refresh opening the Gradle tool window through the `View / Tool Windows / Gradle` menu and clicking the "Refresh all Gradle Projects" button in the tool window that opens.

Run the app and verify that everything is working.

Now that you have the Kotlin plugins installed in Android Studio and set up your Gradle scripts, you should be able to start coding in Kotlin. In the next part, we'll look at converting `MainActivity` to Kotlin and adding the views we need for the photo frame.
