---
layout: post
title: "Android Photoframe in Kotlin: Part 4 - Handling power connect and disconnect"
excerpt: "In this post, We'll add the code to launch the photo frame automatically when your device is connected to power"
tags: [code, kotlin, java, android, photoframe]
modified: 2015-10-12
comments: true
---

// Update: I've posted the [source code](https://github.com/JohnBrainard/KotlinPhotoFrame/) for this series on Github.

We now have a pretty functional photo frame app for our Android device. The only thing left to do to make it useful is to make sure it doesn't shut off on us. To do that, we're going to add a `BroadcastReceiver` to listen to power connect/disconnect events so the app knows whether or not to disable screen timeouts. We're also going to use that `BroadcastReceiver` to launch the app when the power cable is connected.

### The `PowerConnectionReceiver` Class

Create a new Kotlin file in the `io.github.johnbrainard.kotlinphotoframe` package called `PowerConnectionReceiver` and enter the following code:

{% highlight kotlin linenos %}

class PowerConnectionReceiver constructor() : BroadcastReceiver() {
	var disconnected: () -> Unit = {}

	constructor(disconnected:()->Unit) : this() {
		this.disconnected = disconnected
	}

	override fun onReceive(context: Context, intent: Intent) {
		when (intent.getAction()) {
			Intent.ACTION_POWER_CONNECTED -> {
				val activityIntent = Intent(context, javaClass<MainActivity>())
				activityIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
				context.startActivity(activityIntent)
			}

			Intent.ACTION_POWER_DISCONNECTED -> {
				disconnected?.invoke()
			}
		}
	}
}

{% endhighlight %}

Notice that this class has a primary, default constructor and an additional constructor. In a moment, we'll be adding a `receiver` configuration to the `AndroidManifest.xml` file. The second constructor takes a lambda expression as a parameter that's called when the power is disconnected. The `MainActivity` will take advantage of this to allow the screen to time out while the app is running.

### Configuring the `PowerConnectionReceiver`

In your `AndroidManifest.xml` file, add the following `receiver` configuration inside the `application` element:

{% highlight xml %}

<receiver android:name=".PowerConnectionReceiver">
	<intent-filter>
		<action android:name="android.intent.action.ACTION_POWER_CONNECTED" />
	</intent-filter>
</receiver>

{% endhighlight %}


### `isPowerConnected` Extension Property

One of my favorite features of Kotlin is its extension methods and properties. They allow you to do many things that would be difficult to do without out them. We'll look at them in more detail in a future post, including decompiled code to see how they're implemented. For now, we're going to create a simple extension property on Android's `Context` class that we can use to determine if the power is connected.

In Android Studio, add a new file to your `io.github.johnbrainard.kotlinphotoframe` package called `utils.kt`. You can call it whatever you like and put it in whatever package you want, but for the sake of this post, we'll choose the given package and name.

{% highlight kotlin linenos %}
package io.github.johnbrainard.kotlinphotoframe

import android.content.Context
import android.content.Intent
import android.content.IntentFilter
import android.os.BatteryManager

val Context.isPowerConnected:Boolean get() {
	val intent = registerReceiver(null, IntentFilter(Intent.ACTION_BATTERY_CHANGED))
	val plugged = intent.getIntExtra(BatteryManager.EXTRA_PLUGGED, -1)
	return plugged == BatteryManager.BATTERY_PLUGGED_AC || plugged == BatteryManager.BATTERY_PLUGGED_USB
}
{% endhighlight %}

### Disabling Screen Timeout

Now that we have everything set up in the `AndroidManifest.xml` file, the `PowerConnectionReceiver` class created and our convenience extension property for checking the power connection state, we'll add the final touches we need to `MainActivity`.

First we're going to declare a new property and set it to a new instance of `PowerConnectionReceiver`. This will be how we reenable the screen timeout when the power is disconnected.

{% highlight kotlin %}
private val powerReceiver = PowerConnectionReceiver() {
	getWindow().clearFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)
}
{% endhighlight %}

Now we need to register that receiver so it can receive events and disable the screen timeout if the power is connected. This is where we get to use the extension property we just created. Add the following at the end of your `onResume()` method.

{% highlight kotlin %}
registerReceiver(powerReceiver, IntentFilter(Intent.ACTION_POWER_DISCONNECTED))

if (isPowerConnected)
	getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)
{% endhighlight %}

And finally, we need to do some cleanup in `onPause()`. We don't want to leave our `PowerConnectionReceiver` registered as a listener when the activity is no longer active. So, add the following line to the end of your `onPause()` method.

{% highlight kotlin %}
unregisterReceiver(powerReceiver)
{% endhighlight %}

### Wrapping Up

At this point you should have a fully functional photo frame app that, with little effort, could be more useful. Some ideas to consider:

* Add text view to display an image title
* Modify the `ImageSource` to retrieve images from your Flickr account
* Add support for multiple image sources and randomly select

You can check out the [Source Code](https://github.com/JohnBrainard/KotlinPhotoFrame/) for the app and compare yours with mine.

I would love to hear your feedback about how this app can be improved and what ideas to implement in future posts.
