# typescript-static-types

## :page_facing_up: 목차
* [Non-Nullable-Types](#non-nullable-types)
* [Control Flow Based Type Analysis](#control-flow-based-type-analysis)
* [Custom Type Guard Functions](#custom-type-guard-functions)
* [Properties and Index Signatures Readonly](#properties-and-index-signatures-readonly)
* [Non-Primitive Types with TypeScript’s object Type](#non-primitive-types-with-typescripts-object-type)
* [never Type](#never-type)
* [Overload](#overload)
* [String Enum](#string-enum)
* [Literal Types](#literal-types)
* [Discriminated Union types](#discriminated-union-types)
* [Infer Types for Rest and Spread Properties](#infer-types-for-rest-and-spread-properties)
* [keyof and Lookup Types](#keyof-and-lookup-types)
* [Mapped Types](#mapped-types)


## Non-Nullable-Types
```typescript
// non-nullable types

function trimAndLower(text: string | null | undefined) {
  if (typeof text === "string") {
      return text.trim().toLowerCase();
  }
  return text;
}

console.log(trimAndLower(" LearnTypeScript.io "));
console.log(trimAndLower(null));
console.log(trimAndLower(undefined));

// ========================================

const container = document.getElementById("app-container")!;
container.remove();
```

## Control Flow Based Type Analysis
```typescript
// flow analysis
function trimAndLower(text: string | null | undefined) {
  // (parameter) text: string | null | undefined
  // text;

  if (typeof text === "string") {
      // (parameter) text: string
      // text;

      return text.trim().toLowerCase();
  }
  // (parameter) text: null | undefined
  // text;

  console.log(text);
}

console.log(trimAndLower(" LearnTypeScript.io "));

let foo: number;

// [ts] Variable 'foo' is used before being assigned.
console.log(foo)

// definite assignment analysis
let bar: number | undefined;
let bar = 42;

// let bar: number
// this will allow autocompletion
bar

console.log(bar)
```

## Custom Type Guard Functions
```typescript
const numbers = [0, 1, 2, [3, 4], 5, [6], [7], 8, [9]];

function isFlat<T>(array: (T | T[])[]): array is T[] {
    console.log(!array.some(Array.isArray));
}

if (isFlat(numbers)) {
    numbers;
}
```

## Properties and Index Signatures Readonly
`readonly` modifier which can be added to a property or index signature declaration. It helps prevent against unintended property assignments.

```typescript
const weekdays: ReadonlyArray<string> = [
  "Sunday",
  "Monday",
  "Tuesday",
  "Wednesday",
  "Thursday",
  "Friday",
  "Saturday"
];

// [ts] Index signature in type 'ReadonlyArray<string>'
// only permits reading.
weekdays[0] = "Fancyday"


// NOTE: This is only compile time protection. 
// TypeScript does not force immutability
console.log(weekdays);
```

## Non-Primitive Types with TypeScript’s object Type
```typescript
const obj: { [key: string]: string } = {};
obj.hasOwnProperty("foo");
obj.foo = "bar";

console.log(obj.foo)
```

## Never Type
`never` is the type of values that never occur. It helps model the completion behavior of functions more accurately and can also be used for exhaustiveness checking.
```typescript
enum ShirtSize {
  XS,
  S,
  M,
  L,
  XL
}

function assertNever(value: never): never {
  // throw Error(`Unexpected value '${value}'`)
  // Adjusted for plunker output
  console.log(Error(`Unexpected value '${value}'`));
}

function prettyPrint(size: ShirtSize) {
  switch (size) {
      case ShirtSize.S: console.log("small");
      case ShirtSize.M: return "medium";
      case ShirtSize.L: return "large";
      case ShirtSize.XL: return "extra large";
      // [ts] Argument of type 'ShirtSize.XS' is
      // not assignable to parameter of type 'never'.
      default: return assertNever(size);
  }
}

prettyPrint(ShirtSize.S)
prettyPrint(ShirtSize.XXL)
```

## Overload
Some functions may have different return types depending on the types of the arguments with which they’re invoked. Using TypeScript’s function overloads, you can create an `overload` for each allowed combination of parameter and return types. 
```typescript
/**
 * Reverses the given string.
 * @param string The string to reverse.
 */
function reverse(string: string): string;

/**
 * Reverses the given array.
 * @param array The array to reverse.
 */
function reverse<T>(array: T[]): T[];

function reverse(stringOrArray: string | any[]) {
    return typeof stringOrArray === "string" 
    ? [...stringOrArray].reverse().join("") 
    : stringOrArray.slice().reverse();
}

// [ts] const reversedString: string 
const reversedString = reverse("TypeScript");
// [ts] const reversedArray: number[]
const reversedArray = reverse([4, 8, 15, 16, 23, 42]);
console.log(reversedString)
console.log(reversedArray)
```

## String Enum
```typescript
const enum MediaTypes {
  JSON = "application/json"
}

// Endpoint updated to for valid response
fetch("https://swapi.co/api/people/1/", {
  headers: {
      Accept: MediaTypes.JSON
  }
})
.then((res) => res.json())
.then(response => {
// console.log added to show example working
 console.log(response.name)
});
```

## Literal Types
A `literal type` is a type that represents exactly one value, e.g. one specific string or number. You can combine literal types with union types to model a finite set of valid values for a variable. 
* String literal types
* Numeric literal types
* Boolean literal types
* Enum literal types

```typescript
//string literal types

let autoComplete: "on" | "off" = "on";
autoComplete = "off";
// string literal types are case sensitive
// the following code will throw an error
// [ts] Type 'ON' is not assignable to type 'autoComplete'
autoComplete = "ON";

// ============================================================
// numeric literal types

type NumberBase = 2 | 8 | 10 | 16;
let base: NumberBase;
base = 2;
// [ts] Type '3' is not assignable to type 'NumberBase'
base = 3;

// ============================================================
// boolean literal types

let autoFocus: true = true;
// [ts] Type 'false' is not assignable to type 'true'
autoFocus = false;

// ============================================================
// enum literal type

enum Protocols {
    HTTP,
    HTTPS,
    FTP
}

type HyperTextProtocol = Protocols.HTTP | Protocols.HTTPS;

let protocol: HyperTextProtocol;
protocol = Protocols.HTTP;
protocol = Protocols.HTTPS;
// [ts] Type 'Protocols.FTP' is not assignable to type 'HyperTextProtocol'
protocol = Protocols.FTP;
```

## Discriminated Union Types
`discriminated union types` (aka `tagged union types`) allow you to model a finite set of alternative object shapes in the type system. The compiler helps you introduce fewer bugs by only exposing properties that are known to be safe to access at a given location.

```typescript
type Result<T> =
| { success: true; value: T }
| { success: false; error: string };

function tryParseInt(text: string): Result<number> {
if (/^-?\d+$/.test(text)) {
    return {
        success: true,
        value: parseInt(text, 10)
    };
}
return {
    success: false,
    error: "Invalid number format"
};
}

const result = tryParseInt("42");

if (result.success) {
result;
console.log(result.value)
} else {
result;
}


// ============================================================

interface Cash {
kind: "cash";
}

interface PayPal {
kind: "paypal";
email: string;
}

interface CreditCard {
kind: "creditcard";
cardNumber: string;
securityCode: string;
}

type PaymentMethod = Cash | PayPal | CreditCard;

function stringifyPaymentMethod(method: PaymentMethod): string {
switch (method.kind) {
    case "cash":
        return "Cash";
    case "paypal":
        return `PayPal (${method.email})`;
    case "creditcard":
        return "Credit Card";
}
}

const myPayment = {
kind: "paypal",
email: "typescript@egghead.io"
}

console.log(stringifyPaymentMethod(myPayment))
```

## Infer Types for Rest and Spread Properties
It automatically infers rest and spread types so that you can use object spread and rest elements in a statically typed manner without having to manually add type annotations.

```typescript
const person = {
  fullName: "Marius Schulz",
  blog: "https://blog.mariusschulz.com",
  twitter: "@mariusschulz"
};

// rest element must be last
const { fullName, ...socialMedia } = person;

console.log(fullName);
console.log(socialMedia.twitter);

// ============================================================

const defaultStyles = {
  fontFamily: "Arial, sans-serif",
  fontWeight: "normal"
};

const userStyles = {
  color: "#111111",
  fontWeight: 700
};

const styles = {
  ...defaultStyles,
  ...userStyles
};

// ============================================================

const todo = {
  text: "Water the flowers",
  completed: false,
  tags: ["garden"]
};

const shallowCopy = { ...todo };
shallowCopy.text = "Mow the lawn";
shallowCopy.tags.push("weekend");

console.log(shallowCopy.text);
console.log(todo.text);
```

## keyof and Lookup Types
The `keyof` operator produces a union type of all known, public property names of a given type. You can use it together with `lookup` types (aka `indexed access types`) to statically model dynamic property access in the type system.

```typescript
interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

const todo: Todo = {
  id: 1,
  text: "Buy milk",
  completed: false
};

function prop<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const id = prop(todo, "id");
const text = prop(todo, "text");
const completed = prop(todo, "completed");

console.log(id)
console.log(text)
console.log(completed)
```

## Mapped Types
`Mapped types` are a powerful and unique feature of TypeScript's type system. They allow you to create a new type by transforming all properties of an existing type according to a given transformation function.

```typescript
interface Point {
  x: number;
  y: number;
}

type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

type Stringify<T> = {
  [P in keyof T]: string;
};

type PartialNullablePoint = Partial<Nullable<Stringify<Point>>>;

let point: PartialNullablePoint;
point = { x: "0", y: "0" };
point = { x: "0" };
point = { x: undefined, y: null };
console.log(point.x)
console.log(point.y)
```

:memo: **참고 자료**   
* [https://egghead.io/courses/advanced-static-types-in-typescript](https://egghead.io/courses/advanced-static-types-in-typescript)

