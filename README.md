# FP Utils
Yes, this is another fp library for js. On the other and, this library doesn't try to compete with libraries like [fp-ts](https://github.com/gcanti/fp-ts/).
This library is a series of utility functions to make the development process easier.

## Motivation
There are a lot of fp oriented libraries for js, but mostly on one hand they try to bring the entire FP world in js (but js/ts is not Haskell), this making harder to follow the flow; or on the other hand implement the fp concepts in object oriented style, where Just and Nothing instances have a private `__value` and a bunch of methods (`map`, `apply`, `isNothing`, `isJust`, ...) and the value is wrapped in this Just wrapper. While mentioned methods are not bad at all, and I even consider them good; the problem is that I find them pretty hard to be integrated in an existing code base, and for developers that are not familiarised with functional programming, hard to reason about.
The **FP Utils** intention is to provide some simple abstractions that can be very easy integrated with an already existing codebase.

## The definition of `bottom`
While the library want to be very lightweight and easy to use in existing codebase, it does come with a very specific behavior. The `undefined` and `null` are considered as same value and both are considered as **bottom** values. This is important to know and take in consideration when using this library, as many codebases tend to consider `null` as value and you may introduce many issues in your codebase using this library in your codebase.

### Solution
The good part is that this definition is centralized in the `Nothing` type definition. So you can fork this repository and just redefine the `Nothing` meaning, by removing the `null` from definition.
In some specific codebases, values like `""` are considered as `Nothing` too, so you can extend the meaning of `Nothing` like you want.

## Where is `Just`?
The library uses intensively the all known `Maybe` monad, but in same time, there is no instance of `Nothing` and `Just`. Any value tht is not [`Nothing`](https://github.com/GheorgheP/fp-utils/blob/master/src/Nothing.ts) (or in other words, is not `undefined` or `null`) is considered as `Just`. No value is wrapped in a `Just` layer.

## Docs

### Nothing

`Nothing` describes the `null` and `undefined` values. In this library they are considered as the same value.

```ts
import { Nothing } from "fp-utitlities/Nothing";

const asNull: Nothing = null;
const asUndefined: Nothing = undefined;
```

### MNothing
`MNothing<T>` is a generic type that describes whenever a value may be `Nothing` or not.

```ts
import { MNothing } from "fp-utitlities/Nothing";

type MString = MNothing<string>

const asNull: MString = null;
const asUndefined: MString = undefined;
const asString: MString = "test";
```

### isNothing
Check is the value is a `Nothing` value

[ [codesandbox](https://codesandbox.io/s/isnothing-xfxmg?file=/src/index.ts) ]
```ts
import { isNothing, MNothing } from "fp-utitlities/Nothing";

const ls: Array<MNothing<number>> = [1, undefined, 3, null, 4];

const emptiesCount = ls.filter(isNothing).length; // 2
```

### isT
Check whenever a potential maybe value is empty or not

[ [codesandbox](https://codesandbox.io/s/ist-prjm9?file=/src/index.ts) ]
```ts
import { isT } from "fp-utitlities/Nothing";

const inc = (a: number): number => a + 1

const ls: Array<number|undefined> = [1, undefined, 3, undefined, 4];
const incremented = ls.filter(isT).map(inc); // [2, 4, 5]
```

### orElse
Return the default value if the provided value is Nothing
Note: the type of the default value needs to be the same as the type of the checked value.

`orElese` is already curried, so you can use it in both full applied or partial applied form.

[ [codesandbox](https://codesandbox.io/s/orelse-yvi30?file=/src/index.ts) ]
```ts
import { orElse } from "fp-utilities";

const ls: Array<string | undefined> = ["😁", undefined, "😁", undefined, "😁"];
const fillWithSmiles = ls.map(orElse("😱"));

console.log(fillWithSmiles); // ["😁", "😱", "😁", "😱", "😁"];
```

### pass
Provide a predicate and a value for it.
If predicate returns `true`, return back that value.
Otherwise, return `undefined`.
Note: `pass` support both simple boolean predicates and [type guards](https://basarat.gitbook.io/typescript/type-system/typeguard).
So you get full type checker support.
`pass` is very handy in combination with `mPipe`, when you want to guard the pipe with a predicate.

`pass` is already curried, so you can use it in both full applied or partial applied form.

[ [codesandbox](https://codesandbox.io/s/pass-2kfzt?file=/src/index.ts) ]
```ts
import { pass, mPipe } from "fp-utilities";

type Mood = "good" | "bad";

const isMood = (s: string): s is Mood => ["good", "bad"].includes(s as Mood);
const isGood = (m: Mood): m is "good" => m === "good";
const isBad = (m: Mood): m is "bad" => m === "bad";

// Read a "good" value from a string
const goodMoodGalsses = mPipe(pass(isMood), pass(isGood));

// Read a "bad" value from a string
const badMoodGalsses = mPipe(pass(isMood), pass(isBad));

console.log(goodMoodGalsses("no mood")); // undefined
console.log(goodMoodGalsses("bad")); // undefined
console.log(goodMoodGalsses("good")); // "good"

console.log(badMoodGalsses("no mood")); // undefined
console.log(badMoodGalsses("good")); // undefined
console.log(badMoodGalsses("bad")); // "bad"
```

### liftA2
Apply a binary function over result of 2 single functions.

[ [codesandbox](https://codesandbox.io/s/lifta2-gke48?file=/src/index.ts) ]
```ts
import { liftA2 } from "fp-utilities";

const sum = (a: number, b: number): number => a + b;
const inc = (a: number): number => a + 1;
const double = (a: number): number => a * 2;

const fn = liftA2(sum, inc, double);

console.log(fn(2, 3)); // 9
```

### mPipe
Apply a binary function over result of 2 single functions.

[ [codesandbox](https://codesandbox.io/s/mpipe-yv40y?file=/src/index.ts) ]
```ts
import { mPipe } from "fp-utilities";

type T = {
  a?: {
    b?: {
      c?: {
        d?: "bingo";
      };
    };
  };
};

const readBingo: (t: T) => "bingo" | undefined = mPipe(
  (v: T) => v.a,
  (v) => v.b,
  (v) => v.c,
  (v) => v.d
);

console.log(readBingo({})); // undefined
console.log(readBingo({ a: {} })); // undefined
console.log(readBingo({ a: { b: {} } })); // undefined
console.log(readBingo({ a: { b: { c: {} } } })); // undefined
console.log(readBingo({ a: { b: { c: { d: undefined } } } })); // undefined
console.log(readBingo({ a: { b: { c: { d: "bingo" } } } })); // "bingo"

```

### match
Provide a series of type guard predicates that satisfies the input,
and a function that will resolve the value if it matches the type guard.
In other words this is a type safe `if else` statement.
In case the provided type guards list doesn't cover the entire input type,
you'll get a type error at compile time.

**Important:** `match` function requires ts `strictFunctionTypes` or `strict` compiler option to be enabled.

[ [codesandbox](https://codesandbox.io/s/match-1jgrn?file=/src/index.ts) ]

Let's pretend that we have a `T` type, that is an union of `A`, `B` and `C`. We need the function `fn` that will return
a distinct string per input `T`, for `A -> "a"`, `B -> "b"`, `C -> "c"`. In case `A`, `B`, `C` are primitive type like `number` or `string`,
we can use `switch` statement to safely rezolve the constraints. But in case `A`, `B`, `C` are more complex types, like products or unions,
`switch` will be a __no go__, and we are forced in using the `if else` statement.

```ts 
type A = { a: string };
const isA = (t: T): t is A => "a" in t;

type B = { b: number };
const isB = (t: T): t is B => "b" in t;

type C = { c: string[] };
const isC = (t: T): t is C => "c" in t;

type T = A | B | C;

const fn = (v: T): string => {
  if (isA(v)) {
    return "a";
  } else if (isB(v)) {
    return "b";
  } else {
    return "c";
  }
};
```


Now let's pretend that we extend the `T` type, by adding the `D` type. Now we run in trouble as we may not know that
somewhere we have a function `fn` that should return a distinct output for each `A`, `B`, `C` or `D` input.
Also the type checker will not inform us at the compile time, so we will endup with a silent bug on production.

Here `match` comes handy. It just uses the power of type guards in order to detect if all cases are treated.

```ts 
import { match } from "fp-utilities";

type A = { a: string };
const isA = (t: T): t is A => "a" in t;

type B = { b: number };
const isB = (t: T): t is B => "b" in t;

type C = { c: string[] };
const isC = (t: T): t is C => "c" in t;

type T = A | B | C;

const fn: (ab: T) => string = match(
  [isA, () => "a"],
  [isB, () => "b"],
  [isC, () => "c"]
);

console.log(fn({ a: "a" })); // "a"
console.log(fn({ b: 1 })); // "b"
console.log(fn({ c: [] })); // "c"
```
Now in case we add the type `D` to `T`, compiler will give us a compile time error, as not all cases are matched.

**Note:** Don't provide type guards that matches entire type, as in this case `match` will work in same way as `if else`
statement.
