---
layout: userguide
title: User Guide
---
## Your first ST-JS class

Now let's start writing some actual JavaScript, and see how ST-JS works.

1. ### Create the HelloWorld class

We will now create a new <span class="InlineCode">HelloWorld</span> class that adds a bit of dynamic
behavior to the form on our page. We are going to write a little piece of JavaScript that displays a dialog
box that says "Hello Bob" when the user types "Bob" in the form field and presses the "Go" button. Here's
the code that does this:

In the `src/main/java/hello` folder of your project, create a file named `HelloWorld.java` with the following content:

{% highlight java %}
package hello;

// this statement imports all of the standard JavaScript global variables and functions
// (such as window, alert, setTimeout, etc...) so that we can use them directly as we
// would do if we were writing JavaScript directly
import static org.stjs.javascript.Global.*;

// This class will compiled to JavaScript code by the ST-JS maven plugin. The generated
// file contains the class declaration in a format the looks very similar to your Java
// source. This makes ST-JS generated JavaScript code very easy to read and debug.
public class HelloWorld {

    // In the generated JavaScript, the main method is executed after the
    // declaration of the class and its methods
    public static void main(String [] args){

        // On page load, bind the click event to the button.
        // Notes:
        //   - The "window" object is a static member of the Global class that
        //     we've imported at the top of the file
        //   - Here we use the nice and short Java 8 Lambda syntax. It is both
        //     shorter and nicer than the JavaScript function(){} syntax and
        //     provides us with type safety on top
        window.onload = onLoadEvent -> {

            // locate the button and text field elements.
            // Notes:
            //   - as you can see Java immediately allows us to know what's the
            //     type of each variable
            //   - If you are using and IDE, try writing some of the lines below
            //     using code completion, you'll notice that the IDE can directly
            //     make relevant suggestion based on the types of the expression
            //   - $get(index) is where ST-JS diverges slightly from plain JavaScript
            //     forms.$get(0) is translated to forms[0]. See the user guide for
            //     more details on that.
            Form form = window.document.forms.$get(0);
            Element button = form.elements.$get("say");
            Input text = form.elements.$get("to");

            // Bind the onClick event to the button, and make it say Hello
            // Notes:
            //   - Here's that Java 8 lambda syntax again. This syntax, together
            //     with method references, is the whole reason for which we
            //     recommend using Java 8 with ST-JS. The alternative is anonymous
            //     classes and, trust me, you don't want to use that if you can
            //     avoid it, because it is very hard to read.
            //   - "alert" is also a static member of the Global class
            button.onclick = onClickEvent -> {
                alert("Hello " + text.value);
                return true;
            };
        };
    }
}
{% endhighlight %}

2. ### Include JavaScript in the page

We've just written our first bit of JavaScript code, but now we still need to tell the browser to load it.

In your `index.html`, add the following `<head>` section:

{% highlight html %}
<head>
    <!-- This file is the ST-JS runtime. ST-JS needs this file to support features like -->
    <!-- inheritance and method references among other things. This file MUST be included -->
    <!-- BEFORE any ST-JS class. -->
    <script src="generated-js/stjs.js" type="text/javascript"></script>

    <!-- This is the actual JavaScript code for our class. By default ST-JS generates exactly -->
    <!-- one JavaScript file per Java source files. See the rest of the user guide to get an -->
    <!-- overview on how you can change that and on what ST-JS can do. Things it can do includes -->
    <!-- Generating everything in one single file, source maps, etc... -->
    <script src="generated-js/org/stjs/hello/HelloWorld.js" type="text/javascript"></script>
</head>
{% endhighlight %}



3. ### Compile and run your code

In the previous steps, we've told you that you could recompile your JavaScript without ever shutting
down your web server. Let's try that out. First of all, make sure that your web server is still
running. You can restart it with `mvn jetty:run` if necessary. Then rebuild
the JavaScript:

{% highlight bash %}
# Incremental build. Only the classes that have changed are re-compiled
mvn compile stjs:generate
{% endhighlight %}

Your HelloWorld page is now updated, just refresh your browser, or navigate again to:
<a href="http://localhost:8080/HelloWorld/index.html">http://localhost:8080/stjs/index.html</a>
and say Hello to anyone you want!

<img src="/hello.jpg">


4. ### You're done

That's the end of our getting started guide. Please check the rest of the user guide for more complete
information on writing code, unit testing it, building it, packaging it, and running it on your target
environment.
