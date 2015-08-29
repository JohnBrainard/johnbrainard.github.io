---
layout: post
title: SAM Conversions in Kotlin (and Java)
excerpt: "How to implement functional interfaces with Kotlin and Java 8."
tags: [code, kotlin, java, lambda]
modified: 2015-8-29
comments: true
---

Some of the benefits of working with Kotlin and Java 8 include not having to litter your code with anonymous inner classes. They're ugly and require too much boilerplate in order to accomplish what should be simple tasks. Figuring out the syntax can be less than trivial when learning the languages or how to use these new features, however.

Kotlin and Java 8 both support SAM conversions, which is essentially using a lambda expression to implement a functional interface as long as the signature of the lambda expression matches the signature of the single method in the interface.

Working with `JdbcOperations` in some Spring code recently, it took me a little while to figure out exactly how to do this in Kotlin. Here's what I figured out:

{% highlight kotlin %}
fun main(args: Array<String>) {
	val jdbc = JdbcOperations()
	val mapper:RowMapper<Person> = RowMapper { r, i -> Person(name=r.get("name")!!, email=r.get("email")!!) }
	val people = jdbc.query("some query", mapper)
}
{% endhighlight %}

The same in Java

{% highlight java %}
void main(String[] args) {
	RowMapper<Person> mapper = (Map<String,String> row, int index) ->
		new Person(row.get("name"), row.get("email"));
	jdbc.query("some query", mapper);
}
{% endhighlight %}

*or*

{% highlight java %}
class Demo {
	public List<Person> runDemo() {
		JdbcOperations jdbc = new JdbcOperations();
		return jdbc.query("some query", this::mapToPerson);
	}

	Person mapToPerson(Map<String, String> row, int index) {
		return new Person(row.get("name"), row.get("email"));
	}
}
{% endhighlight %}

Read more on [SAM Conversions](http://kotlinlang.org/docs/reference/java-interop.html#sam-conversions) on [kotlinlang.org](http://kotlinlang.org/).
