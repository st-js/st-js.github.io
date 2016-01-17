---
title: 2.5. Object literals and value types
---

Javascript provides a very simple way to build objects that are plain key-value stores and are not members of any
defined class: Object literals. ST-JS allows you to use the JavaScript object literal notation in two different ways:




## JSCollections.$map()

If the type of the value of every in the object is the same, if you don't particularly need to pay attention to
the exact name of each property, or if you need to support exotic property names, you can use the `$map()` method
of `org.stjs.javascript.JSCollections`.

In Java it returns a an `org.stjs.javascript.Map`. In JavaScript, it is translated to a simple object literal.

<div class="grid_6 alpha">
{% highlight java %}
// Java
Map<String, SomeType> map = $map()
Map<String, SomeType> map = $map("k1", v1, "k2", v2)
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript
var map = {}
var map = {"k1": v1, "k2": v2}
{% endhighlight %}
</div>
<div class="clear"></div>




## @SyntheticType

On the other hand, if the values of your properties can be of different types or if your property names are well defined
and you want to make all of this information explicit in your API without incurring the overhead of using a real class
instance, you may use the `@SyntheticType` annotation from the package `org.stjs.javascript.annotation`. As it's name
indicates, `@SyntheticType` can be used to annotate types only.

This annotation has several major effects:

 - The annotated type will not be translated to JavaScript at all, and will simply not exist at runtime.
 - The annotated type may not contain any methods or constructors (which makes sense since the type doesn't exist at
 runtime anyway)
 - The only waay to create instance of the annotated type is to use the Java "double braces" syntax (anonymous inner
 class with an instance initializer block)
 - Within the "double braces" block, only assignment statements may be used
 - All usages of the double brace syntax will be translated to JavaScript object literals

Here is an example that demonstrates all of these points:

<div class="grid_6 alpha">
{% highlight java %}
// Java (declaration)
@SyntheticType
public class AjaxParams {
    public boolean async;
    public String url;
}

// Java (usage)
public class Something {
    public Something(){
        $.ajax(new AjaxParams(){ {
            async = false;
            url = "http://example.com";
        } });
    }
}
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript (declaration)

// nothing... not generated at all...




// JavaScript (usage)

Something = function(){
    $.ajax({
        async : false,
        url : "http://example.com"
    });
}

{% endhighlight %}
</div>
<div class="clear"></div>

This pattern is extensively used in modern JavaScript libraries to pass "Option objects" or "Parameter objects" to
functions. In fact the example above is a simplified version of the signature of the jQuery `ajax()` method and its
associated option object.