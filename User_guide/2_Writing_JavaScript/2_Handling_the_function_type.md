---
layout: userguide
title: Handling the JavaScript Function type
---

## Using the Callback and Function interfaces

JavaScript is a language that treats functions as first class objects. In JavaScript, you can easily manipulate
references to function instances, assign them to variables, invoke the functions directly from those variables,
etc..

Java, on the hand, doesn't really have a function type. Instead, it has interfaces with a single method. Since
Java 8, those interfaces as referred to as "functional interfaces".

In order to support the JavaScript function type in a convenient way, ST-JS introduces two families of Java
functional interfaces that are sufficient to cover most use cases.

The first family represent JavaScript callbacks that take `n` arguments and don't
return a value. The corresponding Java interfaces are the various
`org.stjs.javascript.functions.Callback<em>n</em>` interfaces, where
`0 <= n <= 4`. Each of these interface takes `n` type
parameters that represent the type of each argument supported by the callback.

The second family represent JavaScript functions that take `n` arguments and
return a value. The corresponding Java interfaces are the various
`org.stjs.javascript.functions.Function<em>n</em>` interfaces, where
`0 <= n <= 4`. Each of these interface takes `n+1` type
parameters that represent the type of each argument supported by the function, and where the last type parameter
defines the return type of the function.


All of those interfaces contains a single method named `$invoke` that, when called,
will be translated directly to JavaScript function call.

Here is an example of usage of those interfaces:

<div class="grid_6 alpha">
{% highlight java %}
// Java
public <IN, OUT> OUT produce(
        Function1<? super IN, ? extends OUT> producer,
        IN input){
    return producer.$invoke(input);
}
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript
produce = function(
        producer,
        input){
    return producer(input);
}
{% endhighlight %}
</div>
<div class="clear"></div>

If you need a callback or function type that takes more than 4 arguments, we recommend that you define your own
functional interface (see "Defining your own Function Type" below).

On top of those interfaces, ST-JS also defines three marker interfaces that are not functional interfaces but can be
useful in cases where the exact types of a function are not known or are un-important:
`Callback`, `Function` and `CallbackOrFunction`




## Using Lambdas as functions

Since Java 8, one of the best ways to pass create callbacks and functions are lambdas. In order to respect the
expected semantics and type of `this` of the Java source code, ST-JS automatically
binds the lambda function to `this` at creation time.

<div class="grid_6 alpha">
{% highlight java %}
// Java
// Lambda expression
Function1<Event, Boolean> handler =
    evt -> true;


// Lambda block with 'this' in the closure
Function1<Event, Boolean> handler2 = evt -> {
    console.print(evt.name);
    this.doSomeThing();
    return false;
};

// calling the single-method
handler.$invoke(event);
handler2.$invoke(event);
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript
// Lambda expression
var handler = function(evt){
    return true;
}

// Lambda block
var handler2 = stjs.bind(this, function(evt) {
    console.print(evt.name);
    this.doSomething
    return false;
});

// calling the single-method
handler(event);
handler2(event);
{% endhighlight %}
</div>
<div class="clear"></div>




## Using method reference as functions

Another nice and short way to create functions with ST-JS is to use the method reference feature introduced in Java
8.

<div class="grid_6 alpha">
{% highlight java %}
// Java
// [1] Bound method reference
Function1<Event, Boolean> handler =
    this::handleEvent;

// [2] Instance method reference
Function1<Event, Boolean> handler2 =
    MyType::handleEventOnInstance;

// [3] Static method reference
Function1<Event, Boolean> handler3 =
    MyType::handleEventStatic;

// [4] Constructor method reference
Function1<Event, MyType> handler4 =
    MyType::new;
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript
// [1] Bound method reference
var handler =
    stjs.bind(this, "handleEvent");

// [2] Instance method reference
var handler2 =
    stjs.bind("handleEventOnInstance");

// [3] Static method reference
var handler3 =
    MyType.handleEventStatic;

// [4] Constructor method reference
var handler4 =
    function(){return new MyType(arguments[0]);}
{% endhighlight %}
</div>
<div class="clear"></div>

* *[1], [2]* both of those method references properly support polymorphism. That is the correct, potentially
overridden method will be called event if the runtime type of the bound object is a subtype of its compile time
type.

* *[1], [2], [4]* A new instance of the function is created each time these types of method references are used.
This can be problem for code relying on the identity of the function instance, like for example event listeners
that can be added and removed. If you need to get the exact same instance of a function and still use method
references, you need to store the reference to the function object in a field or variable and use that variable
instead of the method reference directly.

* *[3]* On the other hand, static method references always point to the exact instance of the method and
polymorphism
isn't supported since static methods are not really inherited in Java.




## Defining your own Function type

For readability purposes, you may want to define your own type of functional interface. Doing so can really help
clarifying the API of your classes that make use of your new custom functional interface.

ST-JS supports this possibility through the
`org.stjs.javascript.annotation.JavascriptFunction` annotation.

All the anonymous implementations of the annotated single-method Java interface
will be generated as a JavaScript anonymous function. All invocations of the method
contained in the annotated interface will be generated as a direct call to that function.
The interface itself will not be generated in Javascript.

<div class="grid_6 alpha">
{% highlight java %}
// Java
@JavascriptFunction
public interface EventHandler {
    public boolean onEvent(Event evt);
}

public class Something {
    public Something(Event event){
        // defining a new anonymous implementation
        EventHandler handler = evt -> console.print(evt.name);

        // calling the single-method
        handler.onEvent(event);
    }
}
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript


// the interface is not generated



Something = function(event){
    // translated as an anonymous function
    var handler = function(evt) { console.print(evt.name); };

    // invoke the function directly
    handler(event);
    
}
{% endhighlight %}
</div>
<div class="clear"></div>

## Using anonymous classes as functions

While we strongly recommend that you use lambdas or method references when creating new functions, there are cases
where this is simply not possible (such as when you are stuck on Java 7 or below). For this reason, ST-JS
translates anonymous inner classes that implement an interface annotated with `@JavascriptFunction` to a simple
JavaScript function.

In this case, ST-JS does not perform automatic binding of `this` in the generated JavaSccript and will instead throw
a compile time error when `this` is used inside of the anonymous class. If you need to reference `this` anyway,
assign it to a local `final` variable, and use that variable instead.

<div class="grid_6 alpha">
{% highlight java %}
// Java
final MyType that = this;
Function1<Event, Boolean> handler =
    new EventHandler(){
        public Boolean onEvent(Event evt){
            that.print(evt.name);
            return true;
        }
    };
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript
var that = this;
handler =

    function(event){
        that.print(evt.name);
        return true;

    };
{% endhighlight %}
</div>
<div class="clear"></div>