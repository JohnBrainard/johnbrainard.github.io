---
layout: post
title: "Kotlin Exposed: Introduction"
excerpt: "Kotlin Exposed is a series of posts that look at Kotlin under the hood. We'll pick a feature of the language, compare it to Java and other JVM languages and see how it's compiled"
tags: [code, kotlin, java, kotlin-exposed]
modified: 2015-09-15
comments: true
---

Kotlin is a great JVM language packed with tons of features that make our jobs easier and more enjoyable. It has features that I would have loved to have seen in Java such as properties, top level functions, properties, companion objects, single line DTO classes and so on.

In this series of posts we're going to look under the hood to see what those features of Kotlin are made of. My method will be to write some simple code in Kotlin and de-compile the generated class files and hopefully learn some things. I'll decompile using a variety of methods in hopes to learn as much as I can. This is a learning process for me as I've never decompiled Java code.

For this post, I looked at the compiled code using [Bytecode Viewer](http://bytecodeviewer.com/).

### Classes

To start this series, we'll take a look at a couple of very simple classes and some ways they differ from Java.

#### `SimpleClass.kt`

{% highlight kotlin linenos %}
package io.github.johnbrainard.exposed

class SimpleClass(var someData:String, someParam:String) {
	fun simpleMethod() {
		println("Hello, World!")
	}
}
{% endhighlight %}

{% highlight java linenos %}
/*
 * Decompiled with CFR 0_102.
 *
 * Could not load the following classes:
 *  io.github.johnbrainard.exposed.SimpleClass
 *  jet.runtime.typeinfo.JetValueParameter
 *  kotlin.io.IoPackage
 *  kotlin.jvm.internal.Intrinsics
 *  kotlin.jvm.internal.KotlinClass
 *  kotlin.jvm.internal.KotlinClass$Kind
 *  kotlin.jvm.internal.Reflection
 *  kotlin.reflect.KClass
 *  org.jetbrains.annotations.NotNull
 */
package io.github.johnbrainard.exposed;

import jet.runtime.typeinfo.JetValueParameter;
import kotlin.io.IoPackage;
import kotlin.jvm.internal.Intrinsics;
import kotlin.jvm.internal.KotlinClass;
import kotlin.jvm.internal.Reflection;
import kotlin.reflect.KClass;
import org.jetbrains.annotations.NotNull;

@KotlinClass(abiVersion=23, kind=KotlinClass.Kind.CLASS, data={"\u001d\u0004)Y1+[7qY\u0016\u001cE.Y:t\u0015\tIwN\u0003\u0004hSRDWO\u0019\u0006\rU>DgN\u0019:bS:\f'\u000f\u001a\u0006\bKb\u0004xn]3e\u0015\r\te.\u001f\u0006\u0007W>$H.\u001b8\u000b\rqJg.\u001b;?\u0015!\u0019x.\\3ECR\f'BB*ue&twMC\u0005t_6,\u0007+\u0019:b[*!!.\u0019<b\u0015\u0011a\u0017M\\4\u000b\u0017\u001d,GoU8nK\u0012\u000bG/\u0019\u0006\fg\u0016$8k\\7f\t\u0006$\u0018M\u0003\u0007tS6\u0004H.Z'fi\"|GM\u0003\u0003V]&$(J\u0003\u0002\u0011\u0003)!\u0001\u0002\u0001\t\u0003\u0015\u0011A\u0011\u0001E\u0002\u000b\t!\u0011\u0001\u0003\u0002\u0006\u0007\u0011\r\u0001\u0002\u0001\u0007\u0001\u000b\u0005A1!B\u0002\u0005\u0006!\u0015A\u0002A\u0003\u0004\t\u000bAI\u0001\u0004\u0001\u0006\u0003!-QA\u0001\u0003\u0005\u0011\u0019)!\u0001\"\u0003\t\n\u0015\u0019AQ\u0001\u0005\t\u0019\u0001!\u0001\u0001\u0004\u0002\u001a\u0005\u0015\t\u0001bA\u0017\u0016\t\u0001g\u0001\u0004B\u0011\u0003\u000b\u0005A9!V\u0002\u000f\u000b\r!A!C\u0001\t\u000b5\u0019AQB\u0005\u0002\u0011\u0015\tR\u0001B\u0004\n\u0003\u0011\u0001Q\"\u0001\u0005\u0006['!\u0001\u0001g\u0004\"\u0005\u0015\t\u00012B)\u0004\u0007\u0011=\u0011\"\u0001\u0003\u0001ky)Q\u0004Br\u00011\u000fij\u0001\u0002\u0001\t\t5\u0011Q!\u0001E\u0004!\u000e\u0001QT\u0002\u0003\u0001\u0011\u0015i!!B\u0001\t\bA\u001b\t!\t\u0002\u0006\u0003!\u0011\u0011kA\u0004\u0005\b%\tA\u0001A\u0007\u0002\u0011\u0015i\u0011\u0001C\u0003"})
public final class SimpleClass {
    public static final /* synthetic */ KClass $kotlinClass;
    @NotNull
    private String someData;

    static {
        $kotlinClass = Reflection.createKotlinClass((Class)SimpleClass.class);
    }

    public final void simpleMethod() {
        IoPackage.println((Object)"Hello, World!");
    }

    @NotNull
    public final String getSomeData() {
        return this.someData;
    }

    public final void setSomeData(@JetValueParameter(name="<set-?>") @NotNull String string) {
        Intrinsics.checkParameterIsNotNull((Object)string, (String)"<set-?>");
        this.someData = string;
    }

    public SimpleClass(@JetValueParameter(name="someData") @NotNull String someData, @JetValueParameter(name="someParam") @NotNull String someParam) {
        Intrinsics.checkParameterIsNotNull((Object)someData, (String)"someData");
        Intrinsics.checkParameterIsNotNull((Object)someParam, (String)"someParam");
        this.someData = someData;
    }
}

{% endhighlight %}

One thing to note here is that Kotlin removes a lot of the boilerplate code that you're required to write in Java. Our 7 lines of Kotlin expands to 40 lines of Java!

The decompiled code is mostly a pretty standard Java class. The `var someData:String` parameter to the `SimpleClass` *constructor* is expanded to getter and setter methods. Class propertiees can be declared and initialized in the *constructor* saving several lines of code if all you need to write is a simple DTO type class.

Another thing to notice is that Kotlin classes are final by default. If you want to be able to extend your classes, you'll have to include the `open` attribute.

Kotlin creates a static `KClass` field which stores a reference to [KClassImpl](https://github.com/JetBrains/kotlin/blob/master/core/reflection.jvm/src/kotlin/reflect/jvm/internal/KClassImpl.kt) if you have *kotlin-reflect.jar* in your classpath. Otherwise, it's an instance of [ClassReference](https://github.com/JetBrains/kotlin/blob/master/core/runtime.jvm/src/kotlin/jvm/internal/ClassReference.kt). A quick scan of the source code for `KClassImpl` reveals that the `KClass` property on Kotlin classes provides convenience methods over the Java reflection api.

The `someData` constructor parameter was not declared as nullable, so Kotlin generated null checks in the constructor as well as the setter method for that property.


#### `SimpleDataClass.kt`

Simply adding the `data` attribute to your class adds a lot of extra functionality for free that would require several lines of boilerplate code in Java. The decompiled java class is 81 lines compared to the 7 lines of Kotlin!

Jetbrains is adding restrictions to data classes in *M13* that are detailed in [this blog post](http://blog.jetbrains.com/kotlin/2015/09/feedback-request-limitations-on-data-classes/). Essentially, data classes will be final and only able to implement interfaces and not extend other classes.

{% highlight kotlin linenos %}
package io.github.johnbrainard.exposed

data class SimpleDataClass(var someData:String, someParam:String) {
	fun simpleMethod() {
		println("Hello, ${someData}")
	}
}
{% endhighlight %}

This is the extra functionality that the *data* attribute gives you in the current milestone (*M12*)

{% highlight java linenos %}
public final class SimpleDataClass {

	// ... removed code we've already seen from SimpleClass.kt

    @NotNull
    public final String component1() {
        return this.someData;
    }

    @NotNull
    public final SimpleDataClass copy(@JetValueParameter(name="someData") @NotNull String someData, @JetValueParameter(name="someParam") @NotNull String someParam) {
        Intrinsics.checkParameterIsNotNull((Object)someData, (String)"someData");
        Intrinsics.checkParameterIsNotNull((Object)someParam, (String)"someParam");
        return new SimpleDataClass(someData, someParam);
    }

    @NotNull
    public static /* synthetic */ SimpleDataClass copy$default(SimpleDataClass simpleDataClass, String string, String string2, int n) {
        if ((n & 1) == 0) return simpleDataClass.copy(string, string2);
        string = simpleDataClass.someData;
        return simpleDataClass.copy(string, string2);
    }

    public String toString() {
        return "SimpleDataClass(someData=" + this.someData + ")";
    }

    public int hashCode() {
        String string = this.someData;
        if (string == null) return 0;
        int n = string.hashCode();
        return n;
    }

    public boolean equals(Object object) {
        if (this == object) return true;
        if (!(object instanceof SimpleDataClass)) return false;
        SimpleDataClass simpleDataClass = (SimpleDataClass)object;
        if (!Intrinsics.areEqual((Object)this.someData, (Object)simpleDataClass.someData)) return false;
        return true;
    }
}
{% endhighlight %}

Kotlin allows you to do [Multi-Declarations](http://kotlinlang.org/docs/reference/multi-declarations.html) and does this by using the `componentN` methods. In `SimpleDataClass`, the `componentN` method is only generated for the `someData` property. If `someParam` were declared with `var` or `val`, `component2` would have been generated.

Kotlin allows you to create copies of your data classes with the option to change the value of one or more properties. It does this by using the generated `copy` methods. The first method constructs an entirely new object if you pass in values for both parameters. The second `copy` method, `copy$default` is generated to handle default parameters. For instance, if we invoke `copy` like the following:

{% highlight kotlin %}
val copy1 = instance1.copy(someParam="foo")
{% endhighlight %}

The generated *java* code will be:

{% highlight java %}
SimpleDataClass copy1 = SimpleDataClass.copy$default((SimpleDataClass)instance1, (String)null, (String)"foo", (int)1);
{% endhighlight %}

We'll look at how default parameters are dealt with in Kotlin in more detail in a future post.

The generated `toString`, `hashCode` and `equals` methods are pretty standard and don't really require any analyzing. It's nice to know that we can get this functionality for free in Kotlin and be able to spend more of our time writing business logic.

#### And so...

This simple example demonstrates just the beginning of what Kotlin buys you in terms of productivity. It can save you dozens of lines of code in just simple classes or hundreds or thousands in larger projects, as well as spare you the headaches of dealing with bugs related to typographical errors. As Kotlin evolves and approaches 1.0, some of these things will change. You can already see some differences if you browse through the Kotlin [source code](https://github.com/JetBrains/kotlin) on Github.

If you found this exercise to be useful, please leave a comment. I'd love to hear your thoughts and suggestions for future blog posts.
