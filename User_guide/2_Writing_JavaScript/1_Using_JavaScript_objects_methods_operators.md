---
layout: userguide
title: User Guide
---
## Using JavaScript objects, methods and operators

In general, you can use JavaScript objects in your Java/ST-JS code in exactly the same way as you would in
JavaScript. You can access fields using the familiar `object.propery` notation, and
invoke methods using `object.method(arg)` notation. There are however a few JavaScript
specific idioms and constructs that cannot be translated as-is into Java that you need to know about.

You will find that most methods that cause ST-JS to translate the source Java code into JavaScript code
that doesn't look exactly like the source are prefixed with the `$` character so that
they are easily identifiable (eg: `$map`, `$array`, `$get`, `$put`, etc...)

### Arrays

In ST-JS the JavaScript `Array` type is mapped to the
`org.stjs.javascript.Array` class. Some array related constructs are used in a
different way in Java and Javascript. The list below lists all of the important cases:

<div class="grid_6 alpha">
{% highlight java %}
// Java
// [1] Constructor with new
new Array<SomeType>()
new Array<SomeType>(length)
new Array<SomeType>(el1, el2, el3)

// [2] Constructor without new
Array()
Array(length)
Array(el1, el2, el3)

// [3] Array literal
$array(el1, el2, el3)

// [4] Read access
arr.$get(index)

// [5] Write access
arr.$set(index, value)

// [6] Read length
arr.$length()

// [7] Set length
arr.$length(newLength)

// [8] Iteration with for-in
for(String i : arr){
    ...
}

// [9] Iteration with forEach
arr.$forEach(val -> ...)
arr.$forEach((val, i, a) -> ...)
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript
// [1] Constructor with new
new Array()
new Array(length)
new Array(el1, el2, el3)

// [2] Constructor without new
Array()
Array(length)
Array(el1, el2, el3)

// [3] Array literal (in JSCollections)
[el1, el2, el3]

// [4] Read access
arr[index]

// [5] Write access
arr[index] = value

// [6] Read length
arr.length

// [7] Set length
arr.length = newLength

// [8] Iteration with for-in
for(var i in arr){
    if (arr.hasOwnProperty(i)) { ... }
}

// [9] Iteration with forEach
arr.forEach(function(val){ ... })
arr.forEach(function(val, i, a){ ... })
{% endhighlight %}
</div>
<div class="clear"></div>

All the methods not shown in the snippet above can be called in exactly the same way as you would in JavaScript.

ST-JS also bundles a full JavaDoc of the `Array` API. If your IDE is setup correctly
you should be able to automatically see the documentation for each method right as you type it.

* *[1]* No differences between Java and JavaScript.

* *[2]* No differences between Java and JavaScript. The `Array` method is a static
member of the `org.stjs.javascript.JSGlobal` class.

* *[3]* `$array()` is a static function declared in the class
`org.stjs.javascript.JSCollections`. ST-JS cannot use native Java array literals
because they yield the wrong data type (`SomeType[]` instead of
`Array&lt;Sometype&gt;`.

* *[4],[5]* The java native arrays are native types that don't define any methods, and are therefore not suitable to
mirror the very rich JavaScript arrays. Unfortunately, the `[]` notation can
ony be used with native arrays in Java. The `$get()` method is the ST-JS way of
working around this impedance mismatch between two languages.

* *[6],[7]* In JavaScript arrays, setting the length property of arrays may have side-effects like truncating the
array. In Java, it is unfortunately impossible to apply side effects when a field is set directly. It is
however possible to do so if the field is set through an accessor method instead of directly. One of the goals
of ST-JS was to be able to run short snippets of client-side code directly on the server in order to avoid code
duplication. The `$length()` methods are the ST-JS way of meeting all of these
constraints at the same time.

* *[8]* The Java form of the for-in loop makes one of the quirks of JavaScript very apparent: The for-in loop on
arrays iterates on the indices rather than on the values, it returns all the indices as Strings, and it also
includes all enumerable prototype properties in the iteration. While this behavior makes sense when considering
arrays as Objects in JavaScript, it is pretty confusing and unexpected. ST-JS solves this problem by making it
clear that the iteration type are Strings, and by making sure that the iteration only happens on array indices
and not on any other prototype properties.

* *[9]* In Java 8 the `Iterable` interface which is implemented by the
`Array` class introduced a new method with a default implementation:
`forEach`. Unfortunately, the `Array` already
contained a `forEach` method since its very first version, back before Java 8
became available, but with a very slightly different signature. Unfortunately, the two signatures are considered
different by the Java compiler but are so similar that they are ambiguous when using lambdas or method
references. This means that lambdas and method references couldn't be used without ugly type casting, partially
defeating their purpose of improving code readability. The `$forEach()` methods
solve this problem.

### Associative Arrays / Maps

ST-JS provides a simple Map-like functionality by exposing bare JavaScript objects as key-value stores. The
advantage of this approach is that the generated JavaScript code can be executed directly without depending on
a specific library. Given the static nature of Java, the natural and dynamic
`object["propertyName"]` notation cannot be used directly. The
`org.stjs.javascript.Map` class can be used instead.

<div class="grid_6 alpha">
{% highlight java %}
// Java
// [1] Construction with Object literal
Map<String, SomeType> map = $map()
Map<String, SomeType> map = $map("k1", v1, "k2", v2)

// [2] Reading
map.$get(key)

// [3] Writing
map.$put(key, value)

// [4] Deleting
map.$delete(key)

// [5] Iteration with for-in
for(String key : map){
    ...
}
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript
// [1] Construction with Object literal
var map = {}
var map = {"k1": v1, "k2": v2}

// [2] Reading
map[key]

// [3] Writing
map[key] = value;

// [4] Deleting
delete map[key]

// [5] Iteration with for-in
for(var key in map){
    if (map.hasOwnProperty(key)) { ... }
}
{% endhighlight %}
</div>
<div class="clear"></div>

* *[1]* `$map()` is a static function declared in the class
`org.stjs.javascript.JSCollections`. Just like in Javascript,
you can only use string literals as keys. Attempting to use anything else results in a compile
time error. Values however may be any kind of expression.

* *[2], [3], [4]* Unfortunately, the `[]` notation can ony be used with native arrays
in Java. On top of that, the JavaScript `$delete()` keyword has no equivalent at
all in Java. The `$get()` and `$put()` methods is
the ST-JS way of working around this impedance mismatch between two languages.

* *[5]* No differences between Java and JavaScript



### undefined

The `undefined` value is a pure JavaScript concept that has no equivalent in Java.
ST-JS simply treats `undefined` as `null`. In our
experience, this simple equivalence is enough to handle undefined correctly. Below are some examples of expressions
that are often used when dealing with `undefined` in different cases:

<div class="grid_6 alpha">
{% highlight java %}
// Java
// [1] Checking for equality
myvar == null

// [2] Checking for type
typeof(myvar) == "undefined"

// [3] Assignment
myvar = undefined
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript
// [1] Checking for equality
myvar == null

// [2] Checking for type
(typeof myvar) == "undefined"

// [3] Assignment
myvar = undefined
{% endhighlight %}
</div>
<div class="clear"></div>

* *[1]* No differences between Java and JavaScript. Just like in JavaScript, it makes to sense to write
`myvar == undefined` since nothing is equal to
`undefined`, including `undefined` itself.

* *[2]* Almost no differences between Java and JavaScript. The only difference is the placement of the parenthesis.
The `typeof` function is a static member of the
`org.stjs.javascript.JSGlobal` class.

* *[3]* No differences between Java and JavaScript. The `undefined` value is a static
member of the `org.stjs.javascript.JSGlobal` class.



### The == and === operators

The Java `==` operator is translated directly to the JavaScript
`==` operator.

It might seem at first that the `===` operator is a more faithful translation of the
intent expressed in Java source, but given that Java lacks the concept of `undefined`
and that `null !== undefined` in JavaScript, using `===`
would leave us with no simple way to guard against `undefined` values.
`typeof` could work, but the condition is rather cumbersome to use and needs to be
combined with a check against `null` anyway.

On top of that, the fact that ST-JS code is type checked before being translated to JavaScript pretty much
guarantees that `===` and `==` are equivalent in the
vast majority of cases. There are a few cases where they aren't (such as when the ST-JS code is being called
incorrectly by external JavaScript code), but we have decided to let the users handle those rare cases manually to
gain in clarity in the majority of common cases.



### Constructor, prototype, Class and referencing properties by name

Every JavaScript object contains the special `constructor` and
`prototype` properties. Those properties are not available in the Java
`Object` definition, but ST-JS provides different ways to access them.

The various properties of JavaScript objects can also always be accessed directly by name. ST-JS also provides
different ways to expose this functionality in Java code.

<div class="grid_6 alpha">
{% highlight java %}
// Java
// [1] Get a property by name
$get(somevar, "property")

// [2] Set a property by name
$put(somevar, "property", value);

// [3] Iterating over all properties
for(String propName : $properties(obj)){
    ...
}

// [4] Referencing a prototype
$prototype(obj)

// [5] Referencing a constructor
$constructor(obj)
obj.getClass()
MyType.class
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript
// [1] Get a property by name
somevar["property"]

// [2] Set a property by name
somevar["property"] = value;

// [3] Iterating over all properties
for(var propName in obj){
    if (obj.hasOwnProperty(propName)) { ... }
}

// [4] Referencing a prototype
obj.prototype

// [5] Referencing a constructor
obj.constructor
obj.getClass()
MyType
{% endhighlight %}
</div>
<div class="clear"></div>

* *[1],[2]* The `$get()` and `$set()` methods are
static members of the `org.stjs.javascript.JSObjectAdapter` class. They work
in exactly the same way as their `Map` equivalent, but operate on any object.

* *[3]* The `$properties()` method is a static member of the
`org.stjs.javascript.JSObjectAdapter` class. It basically just casts any object
to a generic STJS `Map&lt;String, Object&gt;` from which you can read properties
using the `Map` API.

* *[4]* The `$prototype()` method is a static member of the
`org.stjs.javascript.JSObjectAdapter` class.

* *[5]* The `$constructor()` method is a static member of the
`org.stjs.javascript.JSObjectAdapter` class.
`$constructor()`, `getClass()` and class literals
are equivalent in terms of what value is actually returned in JavaScript, but only
`$constructor()` is really type safe. In some cases, it might still be preferable
to use the class literal or`getClass()` method to better convey intent in your
code, but if you do so please keep in mind that `java.lang.Class` is not currently
mapped my ST-JS and that any methods you call on this class will fail at runtime.



### The || operator

In javascript, the `||` operator can be used to return the first truthy value of a
list. In Java, that same operator can only be applied to boolean values and will always return a boolean. In most
cases that is enough, but in some cases the JavaScript behavior is desirable. In those cases, you can use the
techniques below:


<div class="grid_6 alpha">
{% highlight java %}
// Java
$or(val1, val2)
$or(val1, val2, val3)
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript
val1 || val2
val1 || val2 || val3
{% endhighlight %}
</div>
<div class="clear"></div>

The `$or()` method is a static member of the
`org.stjs.javascript.JSObjectAdapter` class.


### Handling un-translatable cases

In some rare cases, the JavaScript code cannot be expressed cleanly (or at all) using the ST-JS API. For these
very special cases, you can use `org.stjs.javascript.JSObjectAdapter.$js()`. This
method only accepts string literals and outputs the content of the string directly inside the generated
JavaScript code. The JS code in the string literal must also be a full expression, and will be surrounded by
parenthesis in the generated code.

<div class="grid_6 alpha">
{% highlight java %}
// Java
Array<String> list =
    $js("Array.prototype.slice.call(arguments, 0)")
{% endhighlight %}
</div>
<div class="grid_6 omega">
{% highlight javascript %}
// JavaScript
var list =
    (Array.prototype.slice.call(arguments, 0))
{% endhighlight %}
</div>
<div class="clear"></div>

Please use `$js()` with great care as it is not type safe. It is also not verified to
be syntactically valid and, if used incorrectly, could cause the generated code to be un-parseable.