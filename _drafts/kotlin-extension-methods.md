---
layout: post
title: "Extension methods in Kotlin"
excerpt: "One of my favorite features of Kotlin is its extension methods and properties. They allow you to do many things that would be difficult to do without out them."
tags: [code, kotlin, java, android, photoframe]
modified: 2015-8-29
comments: true
---


### Kotlin Extension methods

One of my favorite features of Kotlin is its extension methods and properties. They allow you to do many things that would be difficult to do without out them. We'll look at them in more detail in a future post, including decompiled code to see how they're implemented. For now, we're going to create a simple extension method on Android's `Resources` class.

In Android Studio, add a new file to your `io.github.johnbrainard.kotlinphotoframe` package called `utils.kt`. You can call it whatever you like and put it in whatever package you want, but for the sake of this post, we'll chose the given package and name.

{% highlight kotlin %}

package io.github.johnbrainard.kotlinphotoframe

import android.content.res.Resources
import android.content.res.TypedArray

fun Resources.getResourceIdArray(id: Int, defaultValue:Int=0): IntArray {
	val typedArray = this.obtainTypedArray(id)
	val intArray: IntArray = IntArray(typedArray.length())
	for (i in 0..typedArray.length()-1) {
		intArray.set(i, typedArray.getResourceId(i, defaultValue))
	}
	return intArray
}

{% endhighlight  %}

The above code adds the `getResourceIdArray` method to the `Resources` classes. The `id` parameter is the resource ID of the images array you created in the previous part and the `defaultValue` parameter is used in the event there isn't a value in the `TyepdArray` at the index and should never be needed.
