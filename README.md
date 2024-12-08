Here's your TypeScript style guide formatted similarly to the provided Next.js style guide:

---

# **⚙️ TypeScript Style Guide**

## **📖 Table of Contents**

### **Core Concepts**

1. [TS Types](#ts-types)
2. [TS Functions](#ts-functions)
3. [TS Variables](#ts-variables)
4. [TS Null & Undefined](#ts-null--undefined)

### **Code Style and Naming**

5. [TS Naming](#ts-naming)
6. [TS React Components](#ts-react-components)
7. [TS Comments](#ts-comments)

### **Best Practices**

8. [TS Source File Structure and Best Practices](#ts-source-file-structure-and-best-practices)

---

# TypeScript

## TS Types

When creating types, we aim to accurately describe our code, which brings several benefits to the codebase:

- **Increased Type Safety**: Catch errors at compile-time by using narrowed types that provide specific information about data shape and behavior.
- **Improved Code Clarity**: Reduce cognitive load with clearer boundaries and constraints on data, making code easier to understand.
- **Easier Refactoring**: Types that are narrow make code changes less risky, allowing for confident refactoring.
- **Optimized Performance**: Narrow types can sometimes help TypeScript generate more optimized JavaScript code.

### Type Inference

As a rule of thumb, explicitly declare a type when it helps narrow it:

```typescript
// ❌ Avoid - Don't explicitly declare a type, it can be inferred.
const userRole: string = "admin"; // Type 'string'
const employees = new Map<string, number>([["Gabriel", 32]]);
const [isActive, setIsActive] = useState<boolean>(false);

// ✅ Use type inference.
const USER_ROLE = "admin"; // Type 'admin'
const employees = new Map([["Gabriel", 32]]); // Type 'Map<string, number>'
const [isActive, setIsActive] = useState(false); // Type 'boolean'

// ❌ Avoid - Don't infer a (wide) type, it can be narrowed.
const employees = new Map(); // Type 'Map<any, any>'
employees.set("Lea", "foo-anything");
type UserRole = "admin" | "guest";
const [userRole, setUserRole] = useState("admin"); // Type 'string'

// ✅ Use explicit type declaration to narrow the type.
const employees = new Map<string, number>(); // Type 'Map<string, number>'
employees.set("Gabriel", 32);
type UserRole = "admin" | "guest";
const [userRole, setUserRole] = useState<UserRole>("admin");
```

### Data Immutability

Majority of the data should be immutable with use of `Readonly`, `ReadonlyArray`.

Using `readonly` type prevents accidental data mutations, which reduces the risk of introducing bugs related to unintended side effects.

When performing data processing always return new array, object etc. To keep cognitive load for future developers low, try to keep data objects small.

As an exception mutations should be used sparingly in cases where truly necessary: complex objects, performance reasoning etc.

#### Examples

```typescript
// ❌ Avoid data mutations
const removeFirstUser = (users: Array<User>) => {
  if (users.length === 0) {
    return users;
  }
  return users.splice(1);
};

// ✅ Use readonly type to prevent accidental mutations
const removeFirstUser = (users: ReadonlyArray<User>) => {
  if (users.length === 0) {
    return users;
  }
  return users.slice(1);
  // Using arr.splice(1) errors - Function 'splice' does not exist on 'users'
};
```

### Return Types

Including return type annotations is highly encouraged, although not required (eslint rule).

Consider benefits when explicitly typing the return value of a function:

- Return values make it clear and easy to understand to any calling code what type is returned.
- In cases where there is no return value, the calling code doesn't try to use the undefined value when it shouldn't.
- Surface potential type errors faster in the future if there are code changes that change the return type of the function.
- Easier to refactor, since it ensures that the return value is assigned to a variable of the correct type.
- Similar to writing tests before implementation (TDD), defining function arguments and return type gives you the opportunity to discuss the feature functionality and its interface ahead of implementation.
- Although type inference is very convenient, adding return types can save TypeScript compiler a lot of work.

### Discriminated Union

If there is only one TypeScript feature to choose, embrace discriminated unions.

Discriminated unions are a powerful concept to model complex data structures and improve type safety, leading to clearer and less error-prone code.

You may encounter discriminated unions under different names such as tagged unions or sum types in various programming languages like C, Haskell, Rust (in conjunction with pattern-matching).

#### Example

```typescript
type Circle = { kind: "circle"; radius: number };
type Square = { kind: "square"; size: number };
type Triangle = { kind: "triangle"; base: number; height: number };

// Create discriminated union 'Shape', with 'kind' property to discriminate the type of object.
type Shape = Circle | Square | Triangle;

// TypeScript warns us with errors in calculateArea function
const calculateArea = (shape: Shape) => {
  // Error - Switch is not exhaustive. Cases not matched: "triangle"
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.size * shape.width; // Error - Property 'width' does not exist on type 'square'
  }
};
```

### Avoid code complexity introduced by flag variables

- Clear code intent, as it becomes easier to read and understand by explicitly indicating the possible cases for a given type.
- TypeScript can narrow down union types, ensuring code correctness at compile time.
- Discriminated unions make refactoring and maintenance easier by providing a centralized definition of related types. When adding or modifying types within the union, the compiler reports any inconsistencies throughout the codebase.
- IDEs can leverage discriminated unions to provide better autocompletion and type inference.

### Template Literal Types

Embrace using template literal types, instead of just (wide) string type.
Template literal types have many applicable use cases e.g. API endpoints, routing, internationalization, database queries, CSS typings ...

#### Examples

```typescript
// ❌ Avoid
const userEndpoint = "/api/usersss"; // Type 'string' - Since typo 'usersss', route doesn't exist and results in runtime error
// ✅ Use
type ApiRoute = "users" | "posts" | "comments";
type ApiEndpoint = `/api/${ApiRoute}`; // Type ApiEndpoint = "/api/users" | "/api/posts" | "/api/comments"
const userEndpoint: ApiEndpoint = "/api/users";

// ❌ Avoid
const homeTitle = "translation.homesss.title"; // Type 'string' - Since typo 'homesss', translation doesn't exist and results in runtime error
// ✅ Use
type LocaleKeyPages = "home" | "about" | "contact";
type TranslationKey = `translation.${LocaleKeyPages}.${string}`; // Type TranslationKey = `translation.home.${string}` | `translation.about.${string}` | `translation.contact.${string}`
const homeTitle: TranslationKey = "translation.home.title";

// ❌ Avoid
const color = "blue-450"; // Type 'string' - Since color 'blue-450' doesn't exist and results in runtime error
// ✅ Use
type BaseColor = "blue" | "red" | "yellow" | "gray";
type Variant = 50 | 100 | 200 | 300 | 400;
type Color = `${BaseColor}-${Variant}` | `#${string}`; // Type Color = "blue-50" | "blue-100" | "blue-200" ... | "red-50" | "red-100" ... | #${string}
const iconColor: Color = "blue-400";
const customColor: Color = "#AD3128";
```

### Type `any` & `unknown`

- `any` data type must not be used as it represents literally “any” value that TypeScript defaults to and skips type checking since it cannot infer the type. As such, `any` is dangerous and can mask severe programming errors.
- When dealing with ambiguous data types, use `unknown`, which is the type-safe counterpart of `any`.
- `unknown` doesn't allow dereferencing all properties (anything can be assigned to `unknown`, but `unknown` isn’t assignable to anything).

#### Examples

```typescript
// ❌ Avoid any
const foo: any = "five";
const bar: number = foo; // no type error

// ✅ Use unknown
const foo: unknown = 5;
const bar: number = foo; // type error - Type 'unknown' is not assignable to type 'number'

// Narrow the type before dereferencing it using:
// Type guard
const isNumber = (num: unknown): num is number => {
  return typeof num === "number";
};
if (!isNumber(foo)) {
  throw Error(
    `API provided a fault value for field 'foo': ${foo}. Should be a number!`
  );
}
const bar: number = foo;

// Type assertion
const bar: number = foo as number;
```

### Type & Non-nullability Assertions

Type assertions (`user as User`) and non-nullability assertions (`user!.name`) are unsafe. Both only silence TypeScript compiler and increase the risk of crashing application at runtime. They can only be used as an exception (e.g. third-party library types mismatch, dereferencing `unknown` etc.) with strong rationale why being introduced into codebase.

#### Example

```typescript
type User = { id: string; username: string; avatar: string | null };
// ❌ Avoid type assertions
const user = { name: 'Nika' } as User;
// ❌ Avoid non-nullability assertions
renderUserAvatar(user!.avatar); // Runtime error

const renderUserAvatar = (avatar: string) => {...};
```

### Type Error

If TypeScript error can't be mitigated, as last resort use `@ts-expect-error` to suppress

it (eslint rule). If at any future point suppressed line becomes error-free, TypeScript compiler will indicate it.
`@ts-ignore` is not allowed, where `@ts-expect-error` must be used with provided description (eslint rule).

#### Example

```typescript
// ❌ Avoid @ts-ignore
// @ts-ignore
const newUser = createUser("Gabriel");

// ✅ Use @ts-expect-error with description
// @ts-expect-error: The library type definition is wrong, createUser accepts string as an argument.
const newUser = createUser("Gabriel");
```

### Type Definition

TypeScript offers two options for type definitions - `type` and `interface`. As they come with some functional differences in most cases they can be used interchangeably. We try to limit syntax difference and pick one for consistency.

All types must be defined with type alias (eslint rule).

#### Example

```typescript
// ❌ Avoid interface definitions
interface UserRole = 'admin' | 'guest'; // invalid - interface can't define (commonly used) type unions

interface UserInfo {
  name: string;
  role: 'admin' | 'guest';
}

// ✅ Use type definition
type UserRole = 'admin' | 'guest';

type UserInfo = {
  name: string;
  role: UserRole;
};

// In case of declaration merging (e.g. extending third-party library types) use interface and disable lint rule.

// types.ts
declare namespace NodeJS {
  // eslint-disable-next-line @typescript-eslint/consistent-type-definitions
  export interface ProcessEnv {
    NODE_ENV: 'development' | 'production';
    PORT: string;
    CUSTOM_ENV_VAR: string;
  }
}

// server.ts
app.listen(process.env.PORT, () => {...});
```

### Array Types

Array types must be defined with generic syntax (eslint rule).

#### Example

```typescript
// ❌ Avoid
const x: string[] = ["foo", "bar"];
const y: readonly string[] = ["foo", "bar"];

// ✅ Use
const x: Array<string> = ["foo", "bar"];
const y: ReadonlyArray<string> = ["foo", "bar"];
```

### Services

Documentation becomes outdated the moment it's written, and worse than no documentation is wrong documentation. The same can be said about types when describing modules your app interacts with (APIs, messaging systems, databases).

For external API services e.g. REST, GraphQL etc. it's crucial to generate types from their contracts, whether it's Swagger, schemas, or other sources (e.g. openapi-ts, graphql-config...). Avoid manually declaring and maintaining them, as it's easy to get them out of sync.

As an exception, manually declare types only when there is truly no documentation provided by the external service.

**[⬆ back to top](#table-of-contents)**

## TS Functions

### General

- Function should have single responsibility.
- Function should be stateless where the same input arguments return the same value every single time.
- Function should accept at least one argument and return data.
- Function should not have side effects, but be pure. Its implementation should not modify or access variable values outside its local environment (global state, fetching etc.).

### Single Object Arg

To keep functions readable and easily extensible for the future (adding/removing args), strive to have a single object as the function argument, instead of multiple args. As an exception, this does not apply when having only one primitive single arg (e.g. simple functions like `isNumber(value)`, implementing currying etc.).

```typescript
// ❌ Avoid having multiple arguments
transformUserInput("client", false, 60, 120, null, true, 2000);

// ✅ Use options object as argument
transformUserInput({
  method: "client",
  isValidated: false,
  minLines: 60,
  maxLines: 120,
  defaultInput: null,
  shouldLog: true,
  timeout: 2000,
});
```

### Required & Optional Args

- Strive to have the majority of args required and use optional sparingly.
- If a function becomes too complex, it probably should be broken into smaller pieces.
- An exaggerated example where implementing 10 functions with 5 required args each is better than implementing one "can do it all" function that accepts 50 optional args.

### Args as Discriminated Union

When applicable, use discriminated union type to eliminate optional args, which will decrease complexity on function API and only necessary/required args will be passed depending on its use case.

```typescript
// ❌ Avoid optional args as they increase complexity of function API
type StatusParams = {
  data?: Products;
  title?: string;
  time?: number;
  error?: string;
};

// ✅ Strive to have majority of args required, if that's not possible,
// use discriminated union for clear intent on function usage
type StatusParamsSuccess = {
  status: 'success';
  data: Products;
  title: string;
};

type StatusParamsLoading = {
  status: 'loading';
  time: number;
};

type StatusParamsError = {
  status: 'error';
  error: string;
};

type StatusParams = StatusParamsSuccess | StatusParamsLoading | StatusParamsError;

export const parseStatus = (params: StatusParams) => {...};
```

**[⬆ back to top](#table-of-contents)**

## TS Variables

### Const Assertion

Strive to use const assertion as `const`:

- Type is narrowed.
- Object gets readonly properties.
- Array becomes readonly tuple.

```typescript
// ❌ Avoid declaring constants without const assertion
const FOO_LOCATION = { x: 50, y: 130 }; // Type { x: number; y: number; }
FOO_LOCATION.x = 10;

const BAR_LOCATION = [50, 130]; // Type number[]
BAR_LOCATION.push(10);

const RATE_LIMIT = 25;
const RATE_LIMIT_MESSAGE = `Max number of requests/min is ${RATE_LIMIT}.`; // Type string

// ✅ Use const assertion
const FOO_LOCATION = { x: 50, y: 130 } as const; // Type '{ readonly x: 50; readonly y: 130; }'
FOO_LOCATION.x = 10; // Error

const BAR_LOCATION = [50, 130] as const; // Type 'readonly [10, 20]'
BAR_LOCATION.push(10); // Error

const RATE_LIMIT = 25;
const RATE_LIMIT_MESSAGE =
  `Max number of requests/min is ${RATE_LIMIT}.` as const; // Type 'Rate limit exceeded! Max number of requests/min is 25.'
```

### Enums & Const Assertion

Const assertion must be used over enum.

```typescript
// .eslintrc.js
'no-restricted-syntax': [
  'error',
  {
    selector: 'TSEnumDeclaration',
    message: 'Use const assertion or a string union type instead.',
  },
],

// ❌ Avoid using enums
enum UserRole {
  GUEST,
  MODERATOR,
  ADMINISTRATOR,
}

enum Color {
  PRIMARY = '#B33930',
  SECONDARY = '#113A5C',
  BRAND = '#9C0E7D',
}

// ✅ Use const assertion
const USER_ROLES = ['guest', 'moderator', 'administrator'] as const;
type UserRole = (typeof USER_ROLES)[number]; // Type "guest" | "moderator" | "administrator"

// Use satisfies if UserRole type is already defined - e.g. database schema model
type UserRoleDB = ReadonlyArray<'guest' | 'moderator' | 'administrator'>;
const AVAILABLE_ROLES = ['guest', 'moderator'] as const satisfies UserRoleDB;

const COLOR = {
  primary: '#B33930',
  secondary: '#113A5C',
  brand: '#9C0E7D',
} as const;
type Color = typeof COLOR;
type ColorKey = keyof Color; // Type "primary" | "secondary" | "brand"
type ColorValue = Color[ColorKey]; // Type "#B33930" | "#113A5C" | "#9C0E7D"
```

### Type Union & Boolean Flags

Strive to use type union variables instead of multiple boolean flag variables.

```typescript
// ❌ Avoid introducing multiple boolean flag variables
const isPending, isProcessing, isConfirmed, isExpired;

// ✅ Use type union variable
type UserStatus = "pending" | "processing" | "confirmed" | "expired";
const userStatus: UserStatus;
```

When maintaining code that makes the number of possible states grow quickly because of flags, there are almost always unhandled states. Utilize discriminated unions.

## TS Null & Undefined

In TypeScript, types `null` and `undefined` can often be used interchangeably. Strive to:

- Use `null` to explicitly state it has no value - assignment, return function type etc.
- Use `undefined` assignment when the value doesn't exist. E.g. exclude fields in form, request payload, database query (Prisma differentiation)

**[⬆ back to top](#table-of-contents)**

## TS Naming

Strive to keep naming conventions consistent and readable, with important context provided, because another person will maintain the code you have written.

### Named Export

Named exports must be used to ensure that all imports follow a uniform pattern (eslint rule).

```javascript
// .eslintrc.js
overrides: [
  {
    files: ["src/pages/**/*"],
    rules: { "import/no-default-export": "off" },
  },
],
```

### Naming Conventions

While it's often hard to find the best name, try to optimize code for consistency and future readers by following conventions:

### Variables

- Locals: Camel case (`products`, `productsFiltered`)
- Booleans: Prefixed with `is`, `has` etc. (`isDisabled`, `hasProduct`)

```javascript
// .eslintrc.js
'@typescript-eslint/naming-convention': [
  'error',
  {
    selector: 'variable',
    types: ['boolean'],
    format: ['PascalCase'],
    prefix: ['is', 'should', 'has', 'can', 'did', 'will'],
  },
],
```

### Constants

- Capitalized (`PRODUCT_ID`)
- Object constants: Singular, capitalized with const assertion.

```typescript
const ORDER_STATUS = {
  pending: "pending",
  fulfilled: true,
  error: "Shipping Error",
} as const;

// If object type exists, use satisfies operator in conjunction with const assertion to ensure the object matches its type.
type OrderStatus = {
  pending: "pending" | "idle";
  fulfilled: boolean;
  error: string;
};

const ORDER_STATUS = {
  pending: "pending",
  fulfilled: true,
  error: "Shipping Error",
} as const satisfies OrderStatus;
```

### Functions

- Camel case (`filterProductsByType`, `formatCurrency`)
- Types: Pascal case (`OrderStatus`, `ProductItem`)

```javascript
// .eslintrc.js
'@typescript-eslint/naming-convention': [
  'error',
  {
    selector: 'typeAlias',
    format: ['PascalCase'],
  },
],
```

### Generics

A generic variable must start with the capital letter T followed by a descriptive name (`TRequest`, `TFooBar`).

```typescript
// ❌ Avoid naming generics with one letter
const createPair = <T, K extends string>(first: T, second: K): [T, K] => {
  return [first, second];
};
const pair = createPair(1, 'a');

// ✅ Name starts with the capital letter T followed by a descriptive name
const createPair = <TFirst, TSecond extends string>(first: TFirst, second: TSecond): [TFirst, TSecond] => {
  return [first, second];
};
const pair = createPair(1, 'a

');

// .eslintrc.js
'@typescript-eslint/naming-convention': [
  'error',
  {
    // Generic type parameter must start with letter T, followed by any uppercase letter.
    selector: 'typeParameter',
    format: ['PascalCase'],
    custom: { regex: '^T[A-Z]', match: true },
  },
],
```

### Abbreviations & Acronyms

Treat acronyms as whole words, with the capitalized first letter only.

```javascript
// ❌ Avoid
const FAQList = ['qa-1', 'qa-2'];
const generateUserURL(params) => {...}

// ✅ Use
const FaqList = ['qa-1', 'qa-2'];
const generateUserUrl(params) => {...}

// In favor of readability, strive to avoid abbreviations, unless they are widely accepted and necessary.

// ❌ Avoid
const GetWin(params) => {...}

// ✅ Use
const GetWindow(params) => {...}
```

## TS React Components

#### Prop Types

React component names should follow the "Props" postfix convention (`[ComponentName]Props` - `ProductItemProps`, `ProductsPageProps`).

#### Callback Props

- Event handler (callback) props are prefixed as `on*` - e.g. `onClick`.
- Event handler implementation functions are prefixed as `handle*` - e.g. `handleClick` (eslint rule).

```javascript
// ❌ Avoid inconsistent callback prop naming
<Button click={actionClick} />
<MyComponent userSelectedOccurred={triggerUser} />

// ✅ Use prop prefix 'on*' and handler prefix 'handle*'
<Button onClick={handleClick} />
<MyComponent onUserSelected={handleUserSelected} />
```

### React Hooks

- Camel case, prefixed as 'use' (eslint rule), symmetrically convention as `[value, setValue] = useState()` (eslint rule).

```javascript
// ❌ Avoid inconsistent useState hook naming
const [userName, setUser] = useState();
const [color, updateColor] = useState();
const [isActive, setActive] = useState();

// ✅ Use
const [name, setName] = useState();
const [color, setColor] = useState();
const [isActive, setIsActive] = useState();

// Custom hook must always return an object:

// ❌ Avoid
const [products, errors] = useGetProducts();
const [fontSizes] = useTheme();

// ✅ Use
const { products, errors } = useGetProducts();
const { fontSizes } = useTheme();
```

**[⬆ back to top](#table-of-contents)**

## TS Comments

In general, try to avoid comments by writing expressive code and naming things what they are.

Use comments when you need to add context or explain choices that cannot be expressed through code (e.g. config files). Comments should always be complete sentences. As a rule of thumb, try to explain why in comments, not how and what.

```javascript
// ❌ Avoid
// convert to minutes
const m = s * 60;
// avg users per minute
const myAvg = u / m;

// ✅ Use - Expressive code and name things what they are
const SECONDS_IN_MINUTE = 60;
const minutes = seconds * SECONDS_IN_MINUTE;
const averageUsersPerMinute = noOfUsers / minutes;

// ✅ Use - Add context to explain why in comments
// TODO: Filtering should be moved to the backend once API changes are released.
// Issue/PR - https://github.com/foo/repo/pulls/55124
const filteredUsers = frontendFiltering(selectedUsers);
// Use Fourier transformation to minimize information loss - https://github.com/dntj/jsfft#usage
const frequencies = signal.FFT();
```

Here's the TypeScript source file structure and best practices formatted with markup for clarity:

---

**[⬆ back to top](#table-of-contents)**

## TS Source File Structure and Best Practices

Files should adhere to a structured format for clarity and maintainability.

### Copyright Information

If necessary, include license or copyright information within a JSDoc at the top of the file.

```javascript
/**
 * @fileoverview Copyright (c) 2024 Your Company. All rights reserved.
 * @license Licensed under the MIT License.
 */
```

### @fileoverview JSDoc

Optionally provide a high-level description of the file's content, dependencies, or usage.

```javascript
/**
 * @fileoverview Description of file content, usage, or dependencies.
 * Detailed description if needed, wrapped as necessary.
 */
```

### Imports

Imports should precede the file's implementation.

#### TypeScript Imports

Four types of imports are commonly used in ES6 and TypeScript:

- Module import: `import * as foo from '...';`
- Named import: `import {SomeThing} from '...';`
- Default import: `import SomeThing from '...';`
- Side-effect import: `import '...';`

```javascript
// Good: Example of imports
import * as ng from "@angular/core";
import { Foo } from "./foo";
import Button from "Button";
import "jasmine";
```

### Import Paths

Use relative paths (`./foo`) for TypeScript imports within the same project to maintain flexibility.

```javascript
// Example of import paths
import { Symbol1 } from "path/from/root";
import { Symbol2 } from "../parent/file";
import { Symbol3 } from "./sibling";
```

### Namespace vs Named Imports

Prefer named imports for clarity and avoid excessively long namespace imports.

```javascript
// Prefer named imports
import { describe, it, expect } from "./testing";
```

### Renaming Imports

Rename imports to avoid naming collisions or improve readability.

```javascript
// Example of renaming imports
import { Item as TableviewItem } from "./tableview";
```

### Exports

Use named exports exclusively for clear and maintainable code.

```typescript
// Example of named exports
export class Foo { ... }
```

### Export Visibility

Export symbols only if they are used outside the module to minimize API surface.

```typescript
// Example of export visibility
export const SOME_CONSTANT = ...
```

### Mutable Exports

Avoid mutable exports to prevent unexpected behavior and maintain code clarity.

```typescript
// Avoid mutable exports
export let foo = 3;
```

### Import and Export Type

Differentiate between importing types and values using `import type` and `export type`.

```typescript
// Example of import type and export type
import type { Foo } from "./foo";
export type { AnInterface } from "./foo";
```

### Use Modules, Not Namespaces

Organize code using ES6 modules (`import` and `export`) instead of namespaces (`namespace`).

```typescript
// Example of using modules instead of namespaces
import { foo } from "bar";
```

---

**[⬆ back to top](#table-of-contents)**
