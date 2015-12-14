---
title: 2.3. Using inheritance
---

Given that JavaScript's prototype based inheritance system cannot be expressed naturally in Java, ST-JS has implemented
class and interface inheritance following the Java model. In most cases, you will find that ST-JS allows you to use all
of the features of the Java type system in your JavaScript code.




## Basic subtyping

Just like in Java, ST-JS lets you subclass your own classes. Since the output of ST-JS is ECMAScript 5, the generated
JavaScript contains a bit more boilerplate code that is necessary to properly implement the full Java inheritance
system. We try to keep it to a minimum, though...

<div class="grid_6 alpha">
{% highlight java %}
// Java

public class Child extends Parent {
    public Child(String name) {
        super(name);
        this.name = name;
    }



    public static int staticField = 1;
    public int instanceField = 2;
    public String name = "default";

    @Override
    public void instanceMethod(String n) {
        super.instanceMethod(n + "-");
    }

    public static void staticMethod() {
        staticField += 1;
    }
}
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript


var Child = function(name) {
   this._super(null, test);
   this.name = name;
}

stjs.extend(Child, Parent, [],
        function(constructor, prototype){
    constructor.staticField = 1;
    prototype.instanceField = 2;
    prototype.name = "default";


    prototype.instanceMethod = function(n) {
        this._super("instanceMethod", n + "-");
    }

    constructor.staticMethod = function() {
        Child.staticField += 1;
    }
}, {}, {});
{% endhighlight %}
</div>
<div class="clear"></div>




## Interfaces and abstract classes

ST-JS supports interfaces and abstract classes just like the rest of the java type system, and will generate the
appropriate types and methods in its JavaScript output.

<div class="grid_6 alpha">
{% highlight java %}
// Java
public interface EventListener {


    public void onEvent(Object event);
}

public class Component implements EventListener {


    public void onEvent(Object event){
       ... // implementation
    }
}

{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript
var EventListener = function() {};
stjs.extend(EventListener, null, [],
        function(constructor, prototype){
    prototype.onEvent = function(event) {};
}, {}, {});

var Component = function() {};
stjs.extend(Component, null, [EventListener],
        function(constructor, prototype){
    prototype.onEvent = function(event) {
        ... // implementation
    };
}, {}, {});
{% endhighlight %}
</div>
<div class="clear"></div>

Interestingly, we thought at first that generating actual JavaScript code for interfaces and abstract methods was just
a waste of time and bandwidth, but as we went on using ST-JS we realized that there was actual, real value in having
this type information available at runtime. With this, you can actually inspect and enumerate members of interfaces,
check if a type is an instance of an interface, generate proper mock objects that mock interfaces and abstract methods,
etc...




## instanceof

The Java `instanceof` keyword is supported by ST-JS and its runtime behavior matches the expected behavior of the Java
version of the keyword.

<div class="grid_6 alpha">
{% highlight java %}
// Java
public void method(Object arg) {
    boolean ok = arg instanceof MyType;
}
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript
prototype.method = function(arg) {
    var ok = stjs.isInstanceOf(arg.constructor, MyType);
};
{% endhighlight %}
</div>
<div class="clear"></div>

When ST-JS translates `instanceof` to JavaScript, it doesn't use the actual corresponding `instanceof` operator of
JavaScript. The JavaScript implementation simply checks if `Class.prototype` is present an object's prototype chain, and
is therefore unable to give the correct result for interfaces because interfaces are references outside of the prototype
chain.

Instead ST-JS uses its own `stjs.isInstanceOf()` function which checks all super types (classes and interfaces)
recursively until either a match is found of the type hierarchy has been fully explored a no match was found.




## Limitations and implementation details

Due to the limitations imposed by the JavaScript prototype inheritance mechanism, there are some features of the Java
type system that we were not able to fully support, or that contain some caveat.

* **Member visibility:** JavaScript doesn't really support the `private`, `protected` and default access modifiers for
fields and methods. While it is possible to emulate `private` members using closures and local variables we have decided
against implementing it that way because the generated code would diverge too much from the Java source. In practice,
this means that all fields and methods are generated as if they were public int the generated JavaScript. This is an
acceptable compromise given that the visibility restrictions are enforced by the Java compiler before the JavaScript
code is generated.

* **Fields and methods share one namespace:** Java effectively maintains separate namespaces for fields and methods.
This allows you to have a field named `isAwesome` and a method with the signature `boolean isAwesome()` in the same
class. JavaScript on the other hand, fields and methods are all simple properties of an object or its prototype, and
each property must have a unique name and therefore fields and methods share the same namespace. This means that the
situation above is not allowed in JavaScript. Again, in an effort to keep the generated JavaScript code as close as
possible to the Java source ST-JS enforces this JavaScript limitation to your Java code, meaning that the situation
above results in a compile time error.

* **Type and super type members share one namespace:** This issue is actually a direct consequence of the previous two.
Given that all properties of a JavaScript object are public and will be looked up through the prototype chain, it is not
possible to have fields or methods in a type that have the same name as a field or method in any of its super types
*unless* they are a valid override of a method from a super type. ST-JS enforces this limitation at compile time.

* **Abstract classes and interfaces:** JavaScript has absolutely no concept of abstract data types. Each property is
either `undefined`, `null` or defined to an `Object` or a `Function`. JavaScript has no way of expressing: "This
property should be provided by all my subtypes, but I don't really define an implementation myself". ST-JS will generate
some code for interfaces and abstract methods, but the resulting generated JavaScript classes are not abstract and could
be instantiated directly by pure JavaScript code. This is an acceptable compromise because the "abstractness" of methods
and types are enforced directly by the Java compiler before the JavaScript code is generated.