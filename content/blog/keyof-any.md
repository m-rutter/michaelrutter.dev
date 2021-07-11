+++
title = "Getting to the bottom of \"keyof any\" in TypeScript"
date = "2021-07-10"
draft = true
[taxonomies]
categories = ["programming", "typescript", "types"]
+++

Learning about the what and why of `keyof any` has taught me a lot about
TypesScript's `any`, `unknown` and `never` types. If you have encountered these
types and want what is hopefully an intuitive an understanding of their
behaviour, then you you might enjoy this dive into `keyof any` and the `Record`
type.

Specifically what I am going to write about is why `keyof any` evaluates to
`string | number | symbol`, and how it relates to `never` and `unknown`.

<!-- more -->

## Utility types

Recently I've been looking at the [utility type][1] delcarations that are
included as part of the TypeScript compiler. These include some very useful type
pairs like `Omit`/`Pick`, `Extract`/`Exclude`, and `Partial`/`Required` that I'm
sure everyone with some TypeScript experience has had or at least will have
cause to use at some point.

A great thing about these types is that they aren't intrinics of the type
system, which means they are defined like [regular user defined types][2]. You
could easily implement them yourself, and they are often **only 1-3 lines of
code**. So they can be a good source of learning about some slightly more
advanced topics in TypeScript.

## The `Record` type

The `Record<K, T>` type is a very useful type for when you want to construct
some kind of key-value pair that maps from some set of properties to a type.
Although JavaScript has had builtin `Map` and `Set` objects for many years now
with arguably a much better API than plain objects for many purposes, it is
still the case that the JavaScript/TypeScript ecosystem largely assumes that
users will use plain objects. That said, even when you can choose between them
it can often be preferable to pick a plain object when all of the key-value
pairs are static and unchanging.

For example, if you wanted to create a map from types of fruit to hex colors you
might write something like this:

```ts
const furitsToColors = {
  orange: "#FFA500",
  apple: "#8DB600",
};
```

A way of defining the type of this object using `Record` could be:

```ts
type Fruit = "orange" | "apple";

type FruitToColorsMap = Record<Fruit, string>;

const furitsToColors: FruitToColorsMap = {
  orange: "#FFA500",
  apple: "#8DB600",
};
```

What `Record<Fruit, string>` effectively means is that if you were to index into
any object of this type with an `"oranage"` or `"apple"`, then you would get
back a `string`.

### Definition and `keyof any`

The definition of `Record` is given as:

```ts
type Record<K extends keyof any, T> = {
  [P in K]: T;
};
```

## stuff

What you might not know is that is that `unknown` and `never` have a special
place in the type system as respectively the "top" and "bottom" types of the
TypeScript type system. The type `any`, which is probaly best know as the type
that essentially turns off type checking, is related to both of these types.
Effectively, it contextually acts as either `unknown` or `never` depending upon
whatever is more permissive. Hopefully we will learn what this means in
practice.

```ts
declare const u: unknown;
declare const n: never;

const a1: unknown = ""; // anything is assignable to unknown
const a2: unknown = u; // including unknown

const b1: never = 1; // type error, nothing is assignable to `never
const b2: never = n; // except `never` itself

const c2: string = u; // type error: unknown is only assignable to unknown
const c3: string = n; // never is assignable to everything

const d: string = 1 as never; // might remind you of `as any`
```

```ts
type A1 = string | unknown; // unknown
type A2 = string & unknown; // string

type B1 = string | never; // string
type B2 = string & never; // never

// any contextually acts as either the top or bottom type to be the most permissive type
type C1 = string | any; // any
type C2 = string & any; // any
```

```ts
declare const f: string | number;

switch (typeof f) {
  case "string":
    break;
  default:
    const shouldneverhappen: never = f; // `number` is a possible missing case
}

switch (typeof f) {
  case "string":
  case "number":
    break;
  default:
    const shouldneverhappen: never = f; // we have accounted for all missing cases
}

if (typeof f === "string") {
  console.log(f);
} else if (typeof f === "number") {
  console.log(f);
} else {
  console.log(f); // f is never because all branches are explored
}
```

```ts
// the grand reveal:

type KN = keyof unknown; // never

type KU = keyof never; // string | number | symbol - ding ding ding
type KA2 = keyof any; // string | number | symbol

// And keyof (A | B) is the same as (keyof A) & (keyof B)
// (A | never) is A
// keyof (A | never) is keyof A
// keyof A == keyof (never | A) == (keyof never) & (keyof A)
// effectively `keyof never` must act as the top type of
// properties and `keyof unknown` the bottom
```

[1]: https://www.typescriptlang.org/docs/handbook/utility-types.html
[2]:
  https://github.com/microsoft/TypeScript/blob/66c3063b06c1a8efb8d25d99db6426478b7dff79/lib/lib.es5.d.ts#L1468
