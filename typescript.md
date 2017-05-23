---
layout: default
title: Moving from ST-JS to TypeScript
---

As you might have noticed, ST-JS is not very actively maintained, the reason is that we are moving to TypeScript.

ST-JS was created before TypeScript even existed, and for a long time was in advance compared to TypeScript. This is no longer true today, the community around TypeScript is really huge, the number of Typings (TypeScript's bridges equivalent) is really enormous. Which makes our effort redundant to Microsoft's effort.

Converting an application from ST-JS to TypeScript is not a particularly complex task but requires to be very concentrated on the whole process.

{:toc}

__Table of contents__
<ul>
    <li><a href="#deciding-on-the-final-output">Deciding on the final output</a></li>
    <li><a href="#webjar-or-npm-module-">Webjar or npm module ?</a></li>
    <li><a href="#the-conversion">The conversion</a>
        <ul>
            <li><a href="#types">Types</a></li>
            <li><a href="#import--export">Import / Export</a></li>
            <li><a href="#array--maps">Array / Maps</a></li>
            <li><a href="#callbacks">Callbacks</a></li>
            <li><a href="#synthetic-types">Synthetic Types</a></li>
            <li><a href="#object-type">Object Type</a></li>
        </ul>
    </li>
    <li><a href="#conclusion">Conclusion</a></li>
</ul>

## Deciding on the final output

ST-JS has only one output way; defining classes in the global scope.

In TypeScript, by default you don't define a single variable in the global scope.

So you have to decide if you wish to adopt EcmaScript/TypeScript's way of defining bundles or keeping global definitions. Here's a quick comparison to help you decide.

__EcmaScript2015+ / TypeScript modules pros and cons__

- _Pro_: your internal classes don't pollute your global scope and don't collision with other classes that might have been defined somewhere else
- _Pro_: your final bundle will be smaller, because all internal classes can be compressed and names can be minified.
- _Con_: it's mandatory to convert all classes in one go. (It might be needed anyway, depending on how your types are linked to each other)

__Global definitions pros and cons__

- _Pro_: In some limited cases, you might be able to gradually move to TypeScript while keeping the rest of your app in ST-JS
- _Con_: Typings for global definitions are really horrible to work with.

## Webjar or npm module ?

ST-JS' preferred way of distribution is webjars. But TypeScript is better off with NPM modules.

While it's possible to distribute an TypeScript project as a webjar, it's very complicated to re-use it in an other project (especially typings) when it's a webjar.

The best solution is to create an npm module first and offer a webjar as well.

## The conversion

### Types

In TypeScript, types are on the right of the statement and are optional.

It's recommended to type at least your public interfaces. The rest can be done by inference a lot.


```java
class Test {
    String name;
    Int count;
 
    String print(String whoami) {
        name = whoami;
    }
}
```

Would give you this in TypeScript:

```typescript
class Test {
    name: string;  // Primitives are lowercase
    count: number; // Int/Double don't exist in JS, everything is "number"
 
    print(whoami: string): string {
        this.name = whoami; // "this." is mandatory, it can't  be inferred
    }
}
```

### Import / Export

Imports and exports in Java are done in only one way, in TypeScript, the EcmaScript 2015+ Spec is used, which is much more flexible.

If you're not familiar with ES2015 imports, I recommend you [read more](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) on the subject.

```java
import com.swissquote.foundation.SomeClass;

class Test {
 
}
```

```typescript
import SomeClass from "./SomeClass"; // Default import
import {someFunction, someConstant} from "./utils"; // named imports

export default class Test {
 
}
```

### Array / Maps

Arrays and maps, because they have special constructors and Types for Java, need to be completely modified for TypeScript

These map and array in ST-JS :

```java
Array<String> test= $array("some", "item", "list")

Map<String, Int> test = $map("key", 2, "other", 3)
test.$put("foo", 4)
```

Are converted to this in TypeScript:

```typescript
let test: string[] = ["some", "item", "list"]

let test: {[key: string]: number} = {key: 2, other: 3}
test.foo = 4;
```

### Callbacks

Callbacks are not very common in Java and are quite cumbersome to implement and use in ST-JS.
On the other hand it's very common in TypeScript and very easy to use.

```java
interface MyType {
    Callback2<Event, Int> onClick;
}

MyType instance;
instance.onClick.$invoke(new Event(), 2);
```

And the same in TypeScript

```typescript
interface MyType {
    onClick: (event: Event, count: number): void ;
}

let instance: MyType;
instance.onClick(new Event(), 2);
```

### Synthetic Types

Synthetic types are types that have a format, but are not actual classes on the client-side, TypeScript has multiple ways of representing them, `interface` is one of them.

This is a Synthetic Type in ST-JS:

```java
@SyntheticType
class MyNonGeneratedType {
    String firstName;
    String lastName;
    Int age;
}
 
MyNonGeneratedType test = new MyNonGeneratedType() {
    {
        firstName = "Stéphane";
        lastName = "Goetz";
        age = 27;
    }
}
```

And in TypeScript:

```typescript
interface MyNonGeneratedType {
    firstName: string;
    lastName: string;
    age: number;
}
 
let test: MyNonGeneratedType = {
    firstName: "Stéphane",
    lastName: "Goetz",
    age: 27
}
```

### Object Type

In TypeScript there is an `object` type but it doesn't represent the same as Java's `Object`

In TypeScript `object` represents everything that isn't `number`, `string`, `boolean`, `symbol`, `null`, or `undefined`.

To get the same effect as java's `Object` you have to use the `any` type

However it's not recommended to use `any`, you might prefer to use combined types or even generics.

## Conclusion

Armed with these tips and tricks, you should be able to convert a codebase of any size from ST-JS to TypeScript 