+++
title = "TypeScript Type Safety: Exploring Overloads & Union Types"
description = "Delving into TypeScript function overloads, union types, and common challenges in search for type safety"
date = "2023-02-22"
draft = false
[taxonomies]
tags = ["typescript", "programming", "types", "java", "type-level-programming"]
+++

In this blog post, we will discuss slightly advanced TypeScript features such as
union types, generics, and conditional types. We will explore how these features
interact when attempting to create type-safe function overloads, a common
challenge faced by developers using TypeScript.

This post was originally notes from an impromptu presentation I gave during a
work "lunch and learn" session a couple of years ago. I've noticed that many
people find this area of TypeScript to be frustrating as they try to combine
different language features, often ending up confused. Let's dive in and learn
how to avoid some common pitfalls.

<!-- more -->

## Some prerequisites

Before we jump in, let's have a quick refresher on three TypeScript features:
union types, generics, and conditional types. Feel free to skip if you're
already familiar with these concepts.

### Union types

Union types in TypeScript allow you to define a type that can be one of several
types.

```ts
interface Yan {
  tan: string;
}

type T1 = "tan" | Yan;

type Tethera = { type: "yan"; value: string } | { type: "tan"; value2: number };

declare const t: Tethera;

// using control flow narrowing to access the correct fields
if (t.type === "yan") {
  t.value.toLowerCase(); // ok
  t.value2.toString(); // compile-time type error - property 'value2' does not exist on type '{ type: "yan"; value: string; }'
}
```

In the example above, `T1` is a union type that can either be a string `"tan"`
or an object of type `Yan`. The `Tethera` type is a union of two different
object shapes, distinguished by their common `type` property. These are
typically called discriminated unions in TypeScript, and elsewhere might be
called tagged unions, disjoint unions, or sum types. TypeScript uses the
language's control flow constructs to "narrow" union types, as demonstrated with
the value `t` above.

### Generics

Generics in TypeScript provide a way to define reusable code that can work with
multiple types. They allow you to create functions that can operate on those
different types while preserving type information of the inputs in its outputs.

```ts
function map<T, U>(type: T[], callback: (t: T) => U): U[] {
  return type.map(callback);
}

const yan = map([2, ""], (value) => value.toString());
//    ^? const yan: string[]
```

In this example, the `map` function takes an array of type `T` and a callback
that maps elements of type `T` to a different type `U`. The result is an array
of type `U`.

### Conditional Types

Conditional types in TypeScript allow you to create new types based on
conditions. They are a powerful way to express complex type relationships by
giving you a form of control flow in type level expressions of the language.

```ts
type T2 = 123 extends number ? true : false;
//   ^? type T2 = true
type T3 = "1" extends 1 ? true : false;
//   ^? type T3 = true

// a type level function that takes a single type as an argument and returns either `true` or `false`
type Printable<T> = T extends string ? true : false;

type T4 = Printable<{ bar: string }>;
//   ^? type T4 = false
```

In this example, `T2` is a conditional type that evaluates to true because `123`
extends `number`, and `T3` evaluates to false because a string `"1"` does not
extend `number`. `T4` evaluates to `false` because the object type
`{bar: string}` does not extend string. The `Printable` type is effectively a
type level function that takes a single type as an argument and return another
type.

## TypeScript function overloads are not type safe

TypeScript supports a form of overloaded functions, which is a common feature in
many languages that allows you to implement multiple related functions with a
common name. When you attempt to use that name to call a function, the
implementation that gets selected is usually determined by the type and arity of
arguments you provide.

Below is an example from Java of a overloaded method called `addToken`. It has
two implementations, one that takes a single argument and another that takes
two. Depending upon the arity of the arugments used to call it, one of the two
implementations is used. The convenience of this feature is that it groups
implementations that conceptually do the same thing, and saves defining seperate
names like `addToken` and `addTokenWithLiteral` for all of the possible
variants:

```java
class Scanner {
    // ...
    // This implementation takes a single arugment of type `TokenType`
	private void addToken(TokenType type) {
	    // ...
	}

    // This implementation takes two arugments of type `TokenType` and `Object`
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

JavaScript does not have function overloads in the sense that Java does, and by
extension TypeScript also lacks function overloads in the conventional sense.
However, it is quite common in JavaScript APIs to have functions that can take a
wild variety of types and arity of arguments. So what TypeScript allows for is
overloading function **signatures**, but with a single implementation:

```ts
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
function yan(tan: string, tethera: number): string;
function yan(tan: number, tethera: string): number;
function yan(tan: string | number, tethera: string | number): string | number {
  return ""; // always returns a string, which satisfies the implementing siganture
}

const num = yan(0, "");
//    ^? const num: number
num.toFixed(0); // runtime error: toFixed is not a function
```

## Can we create our own type-safe function overloads?

TypeScript function overloads are somewhat disappointing because the type
checker does not actually check our work. But is it possible to create a
function signature that does check our work? After surveying the features of
unions and control flow based narrowing, conditional types, and generics, many
users of the language construct something along these lines:

[Playground link](https://www.typescriptlang.org/play?ssl=40&ssc=1&pln=31&pc=1#code/C4TwDgpgBAmghgOygXigbyqSAuKAiERPAbijDhABsB7OAE1wwDc5KBXCXAZ2ACcBLBAHMoAXzHEAUFmgAVRCnSZwnfMCITpKqLIjAAFhF5xFaSVGU41ew8ZLmyFGvUZQW7VQCNq1ShESk7hxcuN6+-ggA2gC6mqJSkgD0iVAmbAj81EjUAGZakFAAggDGwJkIsioA6vwGAApOtHSK8EgAPjoKHboGRnBSMkWl5ZUFqCVlWaMQNfWN9JF4MnjRCYMAkgg5RgBivNQAthMjKgA8slAQAB7AEAh0XEOTFSoAfIoAojfGpacOx1NqrV9A0qE0ADQODAyXAXUSSV5rbRfOAHMB+RSbba8PaHAEvSCnAhERFJFIWAB6AH5JJI6BBipQ4LxoDl0sMslAhHp8edLjc7g8nidIK8ABQOGE6SQASlwWN2+yOHIJEHOpMkxSyPFSKoAjIpucB8WLiQg8DKpMkLJSaVqEDq4CqAEyGnkq03qc2Wsk2qDUzXa4C654AZjdxo9SxsfQtVvJttpbIQKq57uezr511u90e+Om4qlsjlUAVOKV+bOsneZgs-ByUDFg2QLfwhG96AcFntOvbuFapksqjNeHBjjBLiUQWHeDEmi7UBZwDYvCQ7dI1umUAA5K1t1B+I8ENRg3AuFx+EIEHBPBjgNQhzuy7jlc9purtwA6BzwyX6fYAO5QAgEBAR8vD7LwpoAHIngeaJ+Acdy3HQcaSPCQA)

```ts
type Yan = { type: "yan"; payload: { value: string } };
type Tan = { type: "tan" };
type Tethera = {
  type: "tethera";
  payload: { value: boolean; values: boolean[] };
};

// a union of
type ActionTypeWithPayload = Yan | Tan | Tethera;
type ActionType = ActionTypeWithPayload["type"];

type InferFromActionType<T extends ActionType> = Extract<
  ActionTypeWithPayload,
  { type: T }
>;

type Example = InferFromActionType<"yan">;
//   ^? type Example = { type: "yan"; payload: { value: string } }

declare function getAction<T extends ActionType>(
  type: T
): InferFromActionType<T>;

const action1 = getAction("yan");
//    ^? const action1: Yan
const action2 = getAction("tan");
//    ^? const action2: Tan
const action3 = getAction("tethera");
//    ^? const action3: Tethera
```

Well this looks very promising! If you inspect the values of `action1`,
`action2`, and `action2` we get completely different return types from the same
function depending upon the input value, and they all appear to be correct. This
looks a lot like function overloading!

As with the previous examples, we have a single implementing function. However,
this function takes a single arugment `T` that is generic over a union of
`ActionType` variants, one of three different string literal types. That generic
value is passed into to a type level function called `InferFromActionType` as
the return type of the function. This in turn is used by the builtin `Extract`
conditional type, which will pick from the union of `ActionTypeWithPayload` the
approatately matching return type given the input arugment. It would appear that
we have successfully recreated typescript function overloading using other
features of the language.

This is the point of frustration for users because when they turn to
implementing the function, they find that they cannot implement its body. The
only way we could implement the body of getAction is if we assert that the
return value is `InferFromActionType<T>`. Asserting this defeats the entire
purpose of this exercise, which was to create a type-safe overload.

```ts
function getAction2<T extends ActionType>(type: T): InferFromActionType<T> {
  if (type === "yan") {
    const yan: Yan = { type: "yan", payload: { value: "" } };

    return yan; // Type 'Yan' is not assignable to type 'InferFromActionType<T>'.
  }

  throw new Error("Not implemented");
}
```

The stumbling block here is that in the presense of generic types, the
evaluation of conditional types is deferred until the generic value is resolved.
The conditional return type of `getAction` will only be evaluated when the
function is called.

In TypeScript you cannot evaluate a conditional type level function like
`InferFromActionType` if you give it a generic arugment because of this fact. A
simple illustration of this limitation can be seen below. Despite the fact that
`T` has a constraint that it extends `string`, the conditional type remains
unevaluated within the body of the function `tan` because `T` is a generic type.

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
might be desirable for consumers but left implementors without assistance.

The lesson I hope you take away is that if you went down this path, you aren't
alone. Many others thought it might be possible to combine these features, but
the language as it stands today does not enable this pattern. However, this
shouldn't discourage you from seeking alternative solutions. So, as some parting
words, I can suggest some advice for moving forward.

### Solution 1: Implement seperate functions

Take a step back. Reconsider the problem the Java case was trying to solve. They
had separate implementations with a common name, which isn't a feature of
JavaScript. As I suggested earlier, the alternative in the Java case is to
simply have different names for each implementation. So the solution is staring
right at us. Have separately named function implementations for each of your
"overloads."

```ts
function createYanAction(): Yan {
  //...
}

function createTanAction(): Tan {
  //...
}

function createTeheraAction(): Tethera {
  //...
}
```

### Solution 2: Sacrifice type safety

If you are, say, a library author, it might be nice to have overloading function
signatures to give your users a pleasant API to use. Here, the advice would be
to accept that you will have to do extra work to ensure that you are satisfying
overloaded signatures. This is a corner of the language where you aren't going
to get much help, but it might be a worthwhile sacrifice for your users.

What I would urge you to consider is to experiment with findind the right
balance between type safety and a user-friendly API. You will alway find
language limitations for what you are trying to do, but that doesn't necessarily
mean you can't find compromises or workaround.
