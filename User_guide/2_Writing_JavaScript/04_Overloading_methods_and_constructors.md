---
title: 2.4. Overloading methods and constructors
---

In some cases, it can be useful to have multiple versions of the same method that accept slightly different arguments.
Some arguments may be optional, or some arguments may be of different types. Java supports this naturally via method
overloading.

However, because all JavaScript objects and their prototypes are not much more than a key-value dictionaries and that
each key can only have one value, JavaScript does not natively support overloading of methods and constructors.
Typically, when a JavaScript function accepts different sets of arguments the first few lines of the function are
dedicated to figuring out exactly which combination of parameters was passed and the behavior of the code is adjusted
accordingly.




## Overloading methods
In an effort for the generated JavaScript no to deviate too much from the original Java source ST-JS also enforces this
limitation, but with a small twist: there can only be one *concrete* method of the same name. In practice, this means
that you may have multiple methods with the same name in the same type as long as *only one of those methods has a
body*. This can be achieved by adding the `native` modifier to the more specific methods, and implementing the kind
of argument detection logic that is common in JavaScript inside of the more general method.

<div class="grid_6 alpha">
{% highlight java %}
// Java
public native void copyTo(Collection dest);
public native void copyTo(Collection dest, Integer from);
public void copyTo(Collection dest, Integer from, Integer to){
    from = from == null ? 0 : from;
    to = to == null ? this.size() : to;
    ... // implementation
}
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript


prototype.copyTo = function(dest, from, to) {
    from = from == null ? 0 : from;
    to = to == null ? this.size() : to;
    ... // implementation
};
{% endhighlight %}
</div>
<div class="clear"></div>




## Overloading constructors
You can also overload constructors, but the behavior is a little bit different because the `native` modifier is not
allowed on constructors. You can however replace the `native` modifier with the `org.stjs.javascript.annotation.Native`
annotation. In Java constructors are also required to have a body, but the body of any constructor that carries the
`@Native` annotation will be ignored.

<div class="grid_6 alpha">
{% highlight java %}
// Java
public class Collection {
    @Native
    public Collection() {
        // anything you put here will be ignored
    }

    @Native
    public Collection(Collection source) {
        // Same for here
    }

    public Collection(Collection source, int from) {
        ... // there's your actual implementation
    }
}
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript



// all of this stuff was ignored....



// that's a lot of whitespace....



var Collection = function(source, from) {
    ... // there's your actual implementation
};
stjs.extend(Collection, null, [], null, {}, {});
{% endhighlight %}
</div>
<div class="clear"></div>




## A word of warning

While method overloading helps you to design nice and clean APIs, it can devolve into a dangerous mess that is
difficult to understand and maintain. For this reason, we strongly suggest you to follow the few simple rules below to
save you some trouble down the line.

* **Do:** Try to stick to optional parameters where the last parameters of the function are the optional ones, like in the
examples above. This strategy for overloading works well because it is fully type safe and doesn't require any special
argument detection or shuffling code.

* **Don't:** Use native types on parameters that are optional (ie: `int`, `double`) or you will quickly run into
problems with null checking and type casting. Stick to reference types (ie: `Integer`, `Double`, etc...) instead.

* **Don't:** Make one of the middle parameters of your method optional. ie: making optional the 2nd argument of a 5
arguments method. This is not type safe and will actually force you to fight the Java compiler and find ugly mechanisms
to defeat its type checking.

* **Don't:** Have different types for an argument at the same index in various overloaded versions. ie: declaring
`get(int)` and `get(String)`. This will also make you fight the Java compiler. In the long run, one way or another, you
always end up being the one who loses.

* **Do:** Avoid overloading altogether and pick a different method name for each flavor of the method if you really
think you must do one of the above