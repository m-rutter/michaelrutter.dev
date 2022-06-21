+++
title = "An unhappy union; function overloading, generics, and conditional types"
date = "2022-03-18"
[taxonomies]
categories = ["programming", "typescript", "types"]
+++
## Quick Primer


### Unions
```ts
interface Foo2 {
    bar: string
}

type T1 = "foo" | Foo2

type DUnion = {type: "foo", value: string} | {type: "bar", value2: number};

type FooVariant = Extract<DUnion, {type: "foo"}>

declare const d: DUnion;

if (d.type === "foo") {
    d;
//  ^?
    d.value.toLocaleLowerCase()
}
```

### Generics
```ts

function map<T, U>(type: T[], callback: (t: T) => U ): U[] {
    return type.map(callback)
}

const a = map([2, ""], (value) => value.toString());
```


### Conditional Types

```ts
type T2 = 321312 extends number ? true : false;
//    ^?

type T3 = "1" extends 1 ? true : false;
//    ^?

```


<!-- more -->

### Conditional Types + Generics == type level functions

```ts

type Printable<T extends number | "foo" | "bar"> = T extends string ? true : false;

type T4 = Printable<1>
//    ^?

type T5 = Printable<"foo">
//    ^?
type T6 = Printable<number>
//    ^?

type A<T, U> = Extract<T, U>
```

### Use Case lets put them all together for: type safe function overloading

```ts
type Foo = { type: "foo"; payload: { value: string } };
type Bar = { type: "bar" };
type Baz = { type: "baz"; payload: { value: boolean; values: boolean[] } };

type ActionTypeWithPayload = Foo | Bar | Baz;

type ActionType = "foo" | "bar" | "baz";
type ActionType2 = "foo" | "bar";

function getAction<T extends ActionType>(type: T): InferFromActionType<T> {

  if (isFoo(type)) {
     const foo: Foo = {type: "foo", payload: {value: ""}}

     return foo;
  }

  throw "";
}

type InferFromActionType<T extends ActionType> = Extract<ActionTypeWithPayload, { type: T }>;

type ab = InferFromActionType<"foo">

function isFoo(x: ActionType): x is "foo" {
    return true
}

const foo = getAction("foo");
//     ^?

const bar = getAction("bar");
//     ^?

const baz = getAction("baz");
//     ^?


function test2<T extends number>(val: T, val2: string | number) {
  type a = T extends number ? true : false;
  //   ^?
  if (typeof val === "number") {
    val;
//    ^?

    type a = T extends number ? true : false;
    //   ^?

    type b = typeof val2 extends number ? true : false;
    //   ^?

    if (typeof val2 === "number") {
      val2;
      // ^?

      type b = typeof val2 extends number ? true : false;
      //   ^?
    }
  }
}

function test<T extends ActionType>(type: T) {
  const fooBarBaz = getAction(type);

  if (fooBarBaz.type === "foo") {
    fooBarBaz.payload.value.toLowerCase();
  }
}


export function getActionfsdsfsd(): Foo {}
export function getActionfsdfs(): Bar {}
export function getActionfsdfsdfs(): Baz {}


function getAction3<T extends ActionType>(type: T): InferFromActionType<T>;
function getAction3(type: ActionType): ActionTypeWithPayload {
  if (type === "foo") {
    return { type: "foo", payload: { value: "" } }; // yay?
  }

  if (type === "bar") {
    return { type: "foo", payload: { value: "" } }; // is not type safe, returning a Foo instead of a Bar
  }

  throw "";
}

const foo3 = getAction3("foo");
//     ^?


function unique<U, T extends Set<T> | Array<U>>(vals: T ) {

    if (vals instanceof Set){
        vals.has(1 as never)
    }

}

```
