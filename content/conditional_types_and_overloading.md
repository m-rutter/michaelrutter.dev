+++
title = "Typesafe Function Overloading in TypeScript"
description = "Common frustrations with no easy solution"
date = "2023-03-19"
draft = false
[taxonomies]
tags = ["typescript", "programming", "types", "java", "type-level-programming"]
+++

In the course of helping other developers new to TypeScript there are various
common patterns of frustration that I encounter. Two broad categories of
problems:

- Type 1: Users get frustrated that certain patterns that are common in some
  JavaScript codebases are difficult to express in a typesafe way in TypeScript.
  Typically this involves some form of runtime reflection, which I'm not going
  to talk about today and is typically a problem for less experienced or
  especially stubborn users (sometime legitmate stubbornness!).
- Type 2: Users understanding TypeScript features in isolation and get confused
  when they try to combine them in ways that seem conceptually natural, but do
  not work as expected. This is often simply because this combination of
  features has not been implemented yet and may never be implemented.

So we are going to talk about a type 2 problem, specifically function
overloading and conditional types in TypeScript, which is an area of the
language that I think lots of people find unsatisfactory and I've seen recreated
countless times. The intersection between these two features appears to be a
common theme from helping people on the offically unoffical TypeScript community
discord server and at work. The motivation is type safety, but the result is
confusing compiler errors.

<!-- more -->

## Some prerequisites

Before we jump in, let's have a quick refresher on three TypeScript features:
union types, generics, and conditional types. Feel free to skip if you're
already familiar with these concepts.

### Union types

Union types in TypeScript allow you to define a type which can be one of several
types.

```ts
interface Yan {
  tan: string;
}

type YanTan =  Yan | "tan";

type Tethera = { type: "yan"; value: string } | { type: "tan"; value2: number };

declare const tethera: Tethera;

// using control flow narrowing to access the correct fields
if (t.type === "yan") {
  // ok
  t.value.toLowerCase();

  // property 'value2' does not exist on type '{ type: "yan"; value: string; }'
  t.value2.toString();
}
```

In the example above, `YanTan` is a union type that can either be a string
literal type `"tan"` or an object of named `Yan`. The `Tethera` type is a union
of two different object types, distinguished by their common `type` property.
This latter example is typically called a discriminated unions in TypeScript,
and elsewhere might be called tagged unions, disjoint unions, or sum types.
TypeScript uses the language's control flow constructs to "narrow" union types,
as demonstrated with the value `tethera` above.

For more on narrowing the typescript
[handbook chapter on narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
is a must read for new users:

### Generics

Generics in TypeScript provide a way to define reusable code that can work with
multiple types. They allow you to create functions or types that can operate on
those different types while preserving type information of the inputs in its
outputs.

```ts
function map<T, U>(type: T[], callback: (t: T) => U): U[] {
  // ...
}

const yan = map(
  //    ^? const yan: string[]
  [2, ""],
  (value) => value.toString(),
  //  ^? (parameter) value: number | string
);
```

In this example, the `map` function takes an array of type `T` and a callback
that maps elements of type `T` to a different type `U`. The result is an array
of type `U`. This is basically the function signature of `Array.prototype.map`.
In our `yan` example we are mapping an array of `number` and `string` to an
array of `string`, and at each step of the way typescript is able to infer the
types of the input and output of the callback function without any addtional
type annotations.

### Conditional Types

Conditional types in TypeScript allow you to create new types based on type
level conditions. They are a powerful way to express complex type relationships
by giving you a form of control flow in type level expressions of the language.
As an ordinary typescript user it is not a feature you will often need to use
and in fact you may never have need to use it. Howerver, it is a feature that
powers a lot of the more clever forms of type inference and type checking that
you see in the language and in many popular TypeScript libraries.

```ts
type Yan = 123 extends number ? true : false;
//   ^? type Yan = true
type Tan = "1" extends 1 ? true : false;
//   ^? type Tan = false
```

In this example, `Yan` evaluates to the literal boolean type `true` because
`123` extends `number`, and `Tan` evaluates to `false` because a string literal
type `"1"` does not extend the `number` literal `1`. Numbers are numbers, and
strings are not numbers.

```ts
// a type level function that takes a single type as an argument
// and returns either the type `true` or the type `false`
type Printable<T> = T extends string ? true : false;

type P = Printable<{ bar: string }>;
//   ^? type P = false
```

`P` evaluates to `false` because the object type `{ bar: string }` is not a
`string`. The `Printable` type is effectively a type level function that takes a
single type as an argument and returns another type. As you can see this is a
very powerful feature and is a large part of why what is often called the "type
level language" of TypeScript in many ways effectively its own small functional
programming language.

Enough about prelimaries, time to get to the meat of the matter.

## TypeScript function overloads are not type safe

TypeScript supports a form of overloaded functions, which is a common feature in
many languages that allows you to implement multiple related functions with a
common name. When you attempt to use that name to call a function, the
implementation that gets selected is usually determined by the type and/or arity
(number of) of arguments you provide.

Before we look at Typescript's version of overloading lets look at a more
conventional example from Java of a overloaded method called `addToken`:

```java
class Scanner {
    // ...
    // This implementation takes a single argument of type `TokenType`
	private void addToken(TokenType type) {
	    // ...
	}

    // This implementation takes two arguments of type `TokenType` and `Object`
	private void addToken(TokenType type, Object literal) {
	    // ...
	}

	private void scanToken() {
        // ...
		switch (c) {
			case '(':
			    // This calls the first implementation
				addToken(TokenType.LEFT_PAREN);
				break;
            // ...
		}
        // ...
	}

	private void string() {
		// ...
		// This calls the second implementation
		addToken(TokenType.STRING, value);
		// ...
	}
}

```

The Java `Scanner` class has two implementations of `addToken`; one that takes a
single argument and another that takes two. Depending upon the arity (number of
arugments) and arugment types, one of the two implementations is selected. The
convenience of this feature is that it groups implementations that conceptually
do the same thing, and saves defining separate names like `addToken` and
`addTokenWithLiteral` for all of the possible variants:

JavaScript does not have function overloads in the sense that Java does, and by
extension TypeScript also lacks function overloads in the conventional sense. At
runtime there is no enforcement that functions that takes `n` number of
arugments must take `n` number of arugments - a fact some people unfamiliar with
JavaScript often find suprising. It is quite common in JavaScript APIs to have
functions that can take a wild variety of types and arity of arguments despite
the absense of overloads. So what TypeScript allows for is overloading function
**signatures**, but with a single implementation body:

```ts
// our Java addToken method example rewritten as function overloads in TypeScript
function addToken(token: TokenType): void;
function addToken(token: TokenType, literal: any): void;
function addToken(token: TokenType, literal?: any): void {
  // ...
}
```

With TypeScript function overloads, we define multiple signatures, but only
implement a single function that must be compatible with all of the
non-implementing signatures. A frustration with this form of overloads is that
the type checker only enforces the signature of the implementing function, and
does not enforce that the implementing function satisfies all of the overloads.
In short, function overloads are not type safe. They were designed primarily to
help describe existing behavior of JavaScript APIs rather than help you
implement new ones.

For example, below has an implementing function that will fail to return the
correct value as indicated by the overload signatures. TypeScript will select
the second signature overload, but the implementing function will return the
wrong data type resulting in a runtime type error.

```ts
// here we have two overloaded function signatures
function yan(tan: string, tethera: number): string;
function yan(tan: number, tethera: string): number;

// the implementation signature must handle both cases from above,
// however, the return type of the implementation is not checked against the signatures above
function yan(tan: string | number, tethera: string | number): string | number {
  // This implementating function body always returns a string,
  // which is incorrect for the second overload signature
  return "";
}

// When using the 'yan' function with these arugments TypeScript
// will select the second overload signature
const num = yan(0, ""); // Expected return type: number
num.toFixed(0); // Runtime error: toFixed is not a function
```

## Can we create our own type-safe function overloads?

TypeScript function overloads are somewhat disappointing because the type
checker does not actually check our work. But is it possible to create a
function signature that does check our work? After surveying the features of
unions and control flow based narrowing, conditional types, and generics, many
users of the language construct something along these lines:

[Playground link](https://www.typescriptlang.org/play?ssl=9&ssc=1&pln=23&pc=2#code/PTAEBUAsEsGdTqAhqALgTwA4FNQBtsA3bPUAMwFcA7AY1WgHsrkqATZUGp16eppUhhxpISVKAC2STPBSxUAJ2hUA5vl7YFAtAw5cFC7LEzdlaodgBQF0AAUlE3tGIBZaQB5wobAA9U2NngAInQkKiDQAB9QINQwoIA+UABeCG8-ANZg0PDQAH5QeSVVUAAuUCoKCQAjTQBuS0sQCBh4FQDNaBpyajpGZiQaGmxMVFlCxTN1fy1SJFlmZUwKcTD2Q1QKBSp4VEhcfUNjUxKpTBx2MgUGCVAGLZ7aPmYAA3toKXpXaRfLVmwaHgkIZHn0mKB2qgAGoCCjYTzpfyBGI5CLRWLxBIACgA1th0OVwABKcrvRxfbBuTCeBINSxcHbiIpmGF4OEpCHYaGw7BYkLxIl1UDNACiPhwdGw6y5W2YFnKzNU9KY8gqVVqClZ7NSkK1vIx4UFwrAYol-mlm22aCw2HKlRqmkazQAcgwAO6gN24MT+CSjHQIP0ECQBcR7XC6nmg56WShPfqc7ls7AASSD2BDVDizwRviRWRR8SiMTi4WxeIJEBJdgcTm+1PASQA3pZQAgyKBcfiUslUvzDaAW222wzVTk9QrJiU+wBNMKgQg8oINYegDay0DjnlC5rgG2gADkipUB4Q8CoDFWsFg0BUVCQ1QIAZsB7JdcpHkbB9boAAvo022gDsu3QHs+1LIIiUHH8RxVMMwgnNUHQUDkABYACYV2HdcrVLPUdzAPdhAPe0NVPRALyvG87wfJ9UF0F83wpKkaW-Nt-x-PZrg9KhsA9EUDAYBQ+VdcQPkwYNQylSCGl-IA)

```ts
// This is a type level function and a conditional type that maps
// a string literal to a corresponding type
type PrimitiveMap<T extends "yan" | "tan"> = T extends "yan" ? string : number;

// This generic function accepts a string literal as an input and returns
// the corresponding mapped from our function `PrimativeMap`
declare function getValue<T extends "yan" | "tan">(key: T): PrimitiveMap<T>;

const stringValue = getValue("yan"); // Expected return type: string
const numberValue = getValue("tan"); // Expected return type: number
```

Well this looks very promising! If you inspect the values of `stringValue` and
`numberValue` we get completely different return types from the same function
depending upon the input value, and they all appear to be correct. This looks a
lot like TypeScript function overloading!

As with the previous examples, we have a single implementing function. However,
this function takes a single argument `T` that is a generic with the constraint
of `"yan" | "tan"`. That generic value is passed into to a type level function
called `PrimitiveMap` as the return type of the function. This conditional type
will pick the appropriately matching return type given the input argument. It
would appear that we have successfully recreated typescript function overloading
using other features of the language.

This is the point of frustration for users because when they turn to
implementing the function, they find that they cannot implement its body. The
only way we could implement the body of getAction is if we assert that the
return value is `PrimitiveMap<T>`. Asserting this defeats the entire purpose of
this exercise, which was to create a type-safe overload.

```ts
// Now we attempt to implement the getValue function
function getValueImplementation<T extends "yan" | "tan">(
  key: T,
): PrimitiveMap<T> {
  if (key === "yan") {
    const yanValue: string = "Yan value";
    return yanValue; // Type 'string' is not assignable to type 'PrimitiveMap<T>'
  }

  if (key === "tan") {
    const tanValue: number = 42;
    return tanValue; // Type 'number' is not assignable to type 'PrimitiveMap<T>'
  }

  throw new Error("Not implemented");
}
```

The stumbling block here is that in the presence of generic types. As of today
evaluation of conditional types is deferred until the generic value is resolved,
which only happens when the function is called. The conditional return type of
`getValueImplementation` will only be evaluated when the function is called and
not when function body is being implemented.

In TypeScript you cannot evaluate a conditional type level function like
`PrimitiveMap` if you give it a generic argument because of this fact. A simple
illustration of this limitation can be seen below. Despite the fact that `T` has
a constraint that it extends `string`, the conditional type remains unevaluated
within the body of the function `tan` because `T` is a generic type.

```ts
type A = string extends string ? true : false;
//   ^? type A = true

function tan<T extends string>(_: T) {
  type B = T extends string ? true : false;
  //   ^? type B = T extends string ? true : false;
}
```

## So what can we do?

In both the built-in function overloads and our experimentation with combining
other language features, we have failed to achieve type-safe overloading
function bodies. In both cases, we were successfully able to create APIs that
might be desirable for consumers but left implementers without assistance.

### Solution 1: Implement separate functions

Take a step back. Reconsider the problem the Java case was trying to solve. They
had separate implementations with a common name, which isn't a feature of
JavaScript. As I suggested earlier, the alternative in the Java case is to
simply have different names for each implementation. So the solution is staring
right at us. Have separately named function implementations for each of your
"overloads."

```ts
function createYan(): Yan {
  //...
}

function createTan(): Tan {
  //...
}
```

Primarily the purpose of Typescript function overloads is to provide a good
experience for users of existing APIs and potentially for library authors. If
you are not in one of those categories, then it might be best to avoid the
complexity of trying to implement function overloads in TypeScript and the
testing burden it might introduce.

### Solution 2: Sacrifice type safety - for the user's benefit

If you are, say, a library author, it might be nice to have overloading function
signatures to give your users a pleasant API to use. Here, the advice would be
to accept that you will have to do extra work to ensure that you are satisfying
overloaded signatures. This will be in the form of extensive testing. This is a
corner of the language where you aren't going to get much compiler help, but it
might be a worthwhile sacrifice for your users.

Addtionally you could make use of
[runtime assertions](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#assertion-functions)
or
[type guards](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)
to pass the function arugments to more specialised functions like in solution 1
example.

So in conclusion I have only paritial solutions to the problem of implementing
typesafe function overloads in TypeScript. So yeah... sorry about that! However,
the more important point was its clear to see why someone might think combining
these features might work and maybe in future versions of TypeScript it might,
though I suspect it might not be a priority and might incur performance or
complexity costs for implementors and users of conditional types.
