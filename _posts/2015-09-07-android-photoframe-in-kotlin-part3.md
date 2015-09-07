---
layout: post
title: "Android Photoframe in Kotlin: Part 3 - Creating the Image Source and Displaying Photos"
excerpt: "This is where the magic begins. We start off with converting `MainActivity` to a Kotlin file!"
tags: [code, kotlin, java, android, photoframe]
modified: 2015-9-7
comments: true
---

Up until this point, we haven't written much code. Everything we've done so far has been to set up the Android application. This time we get to have a bit of fun. In this part, we're going to:

1. add the images that we'll display in our photo frame
1. implement the image source to encapsulate the image rotation logic
1. add the transition between images
1. and implement the timer code

### Adding the Images

For the purpose of this app, I found a small handful of images on [publicdomainpictures.net](http://www.publicdomainpictures.net). It came up in a search for public domain images, so that's what we're going with. You're free to get your images from wherever you'd like. It doesn't matter. The larger they are, the more likely you are to run into out of memory errors, however. There is no need for them to be any larger than the resolution of your device. We can address ways to deal with out of memory errors when loading images in a future post.

To add the images we'll use for this app,

1. place your images in the `/app/src/main/assets` folder. *Create the folder if it doesn't already exist*
1. Create a new values xml file. You can do this through the *File / New / Android Resource File*
	* Enter `values` in the *File Name* field
	* Select `Values` from the *Resource Type* field
	* Leave the rest of the fields set to their defaults
1. Add an entry for each image you placed in your `assets` folder. Your `values.xml` file should look something like this:

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
	<string-array name="images">
		<!-- http://www.publicdomainpictures.net/view-image.php?image=16456&picture=dark-blue-sky -->
		<item>dark_blue_sky.jpg</item>

		<!-- http://www.publicdomainpictures.net/view-image.php?image=1651&picture=morning-sun&large=1 -->
		<item>morning_sun.jpg</item>

		<!-- http://www.publicdomainpictures.net/view-image.php?image=16457&picture=orange-sky&large=1 -->
		<item>orange_sky.jpg</item>

		<!-- http://www.publicdomainpictures.net/view-image.php?image=8408&picture=pamukkale-travertine -->
		<item>pamukkale_travertine.jpg</item>

		<!-- http://www.publicdomainpictures.net/view-image.php?image=25509&picture=september-in-the-forest -->
		<item>september_in_the_forest.jpg</item>
	</string-array>
</resources>
{% endhighlight %}


### Our `ImageSource`

Our image source is about as simple as it gets. It takes an array of image names from the assets folder. Each call to `nextImage()` returns the next image name in the sequence. When the last image is retrieved from the collection, the index pointer is reset to `0`.

I added mine to the `io.github.johnbrainard.kotlinphotoframe` package with the name `ImageSource.kt`.

{% highlight kotlin %}

class ImageSource(images: Array<String>) {
	val images: Array<String> = images
	var index = 0

	fun nextImage(): String {
		val image = images.get(index++)

		if (index >= images.size())
			index = 0

		return image
	}
}

{% endhighlight %}

### Adding the timer

We're not going to use a timer object. We're going to create a handler that we can use to post delayed messages to. This is how we're going to implement the timer. Everytime the callback is invoked, a new `postDelayed` message is sent to the handler.

In `MainActivity`, add the following fields:

{% highlight kotlin %}

val handler = Handler()
val callback = Runnable { selectNextImage() }

var currentDrawable:Drawable? = null

{% endhighlight %}

The `handler` field is the instance of `Handler` we'll be posting the messages to. The `callback` `Runnable` will be invoked by `handler` at the specified intervals, which will call our `selectNextImage()` method. The `currentDrawable` field is a reference to the drawable that the `ImageView` is displaying. We'll need to use this in the `TransitionDrawable` for the transition animation.

[Runnable](https://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html) is a SAM type (*Single Abstract Method*), which means function literals can be [converted automatically](http://kotlinlang.org/docs/reference/java-interop.html#sam-conversions) to an implementation of the class or interface. This is especially useful on Android where many UI elements use single method interfaces as event listeners. Function literals can reduce the complexity of your code considerably and clean up the mess of anonymous inner classes that are common in Android code.

### Drawing the images

The `seletNextImage()` method is going to be responsible for retrieving the next image from the `ImageSource`, creating the `TransitionDrawable`, updating the `ImageView` and posting the next delayed callback.

{% highlight kotlin linenos %}

fun selectNextImage() {
	val imageName = images?.nextImage() ?: return

	thread(name="image-loader") {
		val drawable = Drawable.createFromStream(getAssets().open(imageName), imageName)

		runOnUiThread {
			if (currentDrawable != null) {
				val transition = TransitionDrawable(arrayOf(currentDrawable, drawable))
				imageView.setImageDrawable(transition)
				transition.startTransition(250)
			} else {
				imageView.setImageDrawable(drawable)
			}

			currentDrawable = drawable
			handler.postDelayed(callback, 10000)
		}
	}
}

{% endhighlight %}

Let's look at the important parts here.

*Line 2*: Because `images` can be null, we need to make null-safe calls to the methods or return from `selectNextImage()`. The [safe call](http://kotlinlang.org/docs/reference/null-safety.html#safe-calls) operator, `?.`, ensures that `nextImage()` is only called if `images` is not null. The other part of this is the [elvis operator](http://kotlinlang.org/docs/reference/null-safety.html#elvis-operator). Typically it's used to assign a default value, but we're using it here to return from the method. I don't like this syntax for returning if a value is null. Return isn't a value we're assigning to `imageName` and this can easily be missed when scanning through the code trying to make changes. I'll always write the code like the following:

{% highlight kotlin %}

if (images == null)
	return

drawable = images!!.nextImage()

{% endhighlight %}

*Line 4*: Because of the simplicity of the application, we can get away with using a raw thread rather than a thread pool. Jetbrains includes better async task management in their [Anko](https://github.com/JetBrains/anko) library which we'll explore in future posts. For now, a simple thread will do so we're not loading the image and performing disk IO on the UI thread.

*Line 7*: Two interesting things to note about this line are *(a)*, once the image is loaded, the rest of the work can be done in the UI thread. And *(b)*, `runOnUiThread` is a [method](http://developer.android.com/reference/android/app/Activity.html#runOnUiThread(java.lang.Runnable)) on the `Activity` class. The `runonUiThread` method takes a [SAM type]({% post_url 2015-08-29-sam-conversions-in-kotlin %}) as its only parameter, which means we can omit the parenthesis and we can pass in a closure and the compiler will do the right thing.

*Line 17*: This is what keeps the images rotating. Instead of having a timer object that makes repeated calls to a callback, we schedule the next call upon the successful completion of the `selectNextImage()` method. There are plenty of discussions on StackOverflow explaining why this is preferred over using a timer object so we won't go into detail here.

### Wrapping up
The last thing we need to do is clean up our handler when the app closed or moved to the background. We'll do that in the `onPause()` method.

{% highlight kotlin linenos %}

override fun onPause() {
	super.onPause()
	handler.removeCallbacks(callback)
}

{% endhighlight %}

Running the app now should result in a functioning photoframe.

A photoframe shouldn't shut off when it's being used so we'll look at disabling the screen timeout in the next post as well as running the app when the charging cable is plugged in.
