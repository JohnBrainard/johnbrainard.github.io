---
layout: post
title: "Android Photoframe in Kotlin: Part 2 - Introducing the Main Views (and Kotlin)"
excerpt: "This is where the magic begins. We start off with converting `MainActivity` to a Kotlin file!"
tags: [code, kotlin, java, android, photoframe]
modified: 2015-09-12
comments: true
---

// Update: I've posted the [source code](https://github.com/JohnBrainard/KotlinPhotoFrame/) for this series on Github.

[Last time]({% post_url 2015-08-30-android-photoframe-in-kotlin-part1 %}), we set up Android Studio, created our app and configured it to use Kotlin. This time, we're going convert the `MainActivity` class to Kotlin using the Kotlin plugin.

### Our first Kotlin

The Kotlin plugin makes it very easy to convert your existing Java code to Kotlin. One of their stated purposes behind Kotlin is to have seamless interoperability with Java, and in my limited experience with Kotlin, Jetbrains has done remarkably well in keeping that goal.

To convert your code to Kotlin,

1. Open the class you want to convert (`MainActivity`) and press `Cmd+Shift+A` or `Ctrl+Shift+A`, depending on your platform, to open the *Find Action* tool
2. Begin typing `Convert to Kotlin`.
3. Select *Convert Java File to Kotlin* in the search results

If all went well, you should have a `MainActivity` class that looks like this:

{% highlight kotlin %}

public class MainActivity : AppCompatActivity() {

	override fun onCreate(savedInstanceState: Bundle?) {
		super.onCreate(savedInstanceState)
		setContentView(R.layout.activity_main)
	}

	override fun onCreateOptionsMenu(menu: Menu): Boolean {
		// Inflate the menu; this adds items to the action bar if it is present.
		getMenuInflater().inflate(R.menu.menu_main, menu)
		return true
	}

	override fun onOptionsItemSelected(item: MenuItem): Boolean {
		// Handle action bar item clicks here. The action bar will
		// automatically handle clicks on the Home/Up button, so long
		// as you specify a parent activity in AndroidManifest.xml.
		val id = item.getItemId()

		//noinspection SimplifiableIfStatement
		if (id == R.id.action_settings) {
			return true
		}

		return super.onOptionsItemSelected(item)
	}
}

{% endhighlight %}

*Note: I ran into a `ClassNotFoundException` when running the first time. Cleaning and building seems to resolve that. I'm investigating the issue to see what the cause is.*


### Adding our Views

We'll come back to the `MainActivity` class shortly. For now, we want to set up our view to display images in the photo frame.

First, let's get rid of the Action Bar and make the app full screen. What good is a photo frame if you're forced to look at the app's UI?

In your `styles.xml` file (find using *Cmd+Shift+O*), change the parent of the the `AppTheme` to `Theme.AppCompat.Light.NoActionBar` and set `android:windowFullScreen` to `true`. Your `AppTheme` style should look like this:

{% highlight xml %}
<!-- Base application theme. -->
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
	<!-- Customize your theme here. -->
	<item name="android:windowFullscreen">true</item>
</style>
{% endhighlight %}

In `activity_main.xml`,

1. Replace the `RelativeLayout` with a `FrameLayout` and remove the padding attributes
2. Remove the `TextView` and add an `ImageView` setting `android:id` to `@id+/imageView`, `android:layout_width` and `android:layout_height` to `match_parent` and `android:scaleType` to `centerCrop`

Your `activity_main.xml` file should now look like this:

{% highlight xml %}
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
				xmlns:tools="http://schemas.android.com/tools"
				android:layout_width="match_parent"
				android:layout_height="match_parent"
				tools:context=".MainActivity">

	<ImageView
		android:id="@+id/imageView"
		android:scaleType="centerCrop"
		android:layout_width="match_parent"
		android:layout_height="match_parent"/>

</FrameLayout>
{% endhighlight %}

Setting `layout_width` and `layout_height` to `match_parent` on the `ImageView` ensures the image view fills the screen. Setting `scaleType` to `centerCrop` will make sure the images are scaled to the image view and cropped just enough on the sides or top and bottom to fill the image view. This will make sure there's no extra padding around the images if they aren't the exact aspect ratio of your device.

### Using our Views

We'll finish this part off by getting a reference to our views.

Back in our `MainActivity` class, we're going to import our `main_activity` view! Yep! we can import our view files and use their contents as if they were resources in our other resource files.

1. Add `import kotlinx.android.synthetic.activity_main.imageView`. This will allow you to use your `ImageView` wherever you'd like in your activity class without having to call `findViewById(R.id.imageView)` and casting it to `ImageView`.

That's it! Once you've called `setContentView()` in `onCreate()`, you can access your views by their ID as if they're members of your activity class. For fun, I added `val imageView = this.imageView` to my code and decompiled the result to see what the Kotlin extension did behind the scenes. It generates the call to `findViewById` for you and caches the result in a `HashMap` local to the `MainActivity` instance. Here's the essential parts of the decompiled output:

{% highlight java %}
public final class MainActivity extends AppCompatActivity
{
  private HashMap _$_findViewCache;

  protected void onCreate(@Nullable @JetValueParameter(name="savedInstanceState", type="?") Bundle savedInstanceState)
  {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    ImageView imageView = (ImageView)_$_findCachedViewById(R.id.imageView);
  }

  public View _$_findCachedViewById(int paramInt)
  {
    if (this._$_findViewCache == null) {
      this._$_findViewCache = new HashMap();
    }
    View localView = (View)this._$_findViewCache.get(Integer.valueOf(paramInt));
    if (localView == null)
    {
      localView = findViewById(paramInt);
      this._$_findViewCache.put(Integer.valueOf(paramInt), localView);
    }
    return localView;
  }

  public void _$_clearFindViewByIdCache()
  {
    if (this._$_findViewCache != null) {
      this._$_findViewCache.clear();
    }
  }
}
{% endhighlight %}

In the next part, we'll add our images, an image source and the transition animations.
