# Why `unknown` is Safer Than `any` in TypeScript: Type Safety & Narrowing Explained

## Introduction

TypeScript-এর সবচেয়ে বড় সুবিধা হলো **type safety** — কোড লেখার সময়ই ভুল ধরে ফেলা। কিন্তু `any` টাইপ ব্যবহার করলে এই সুরক্ষা পুরোপুরি বন্ধ হয়ে যায়। অন্যদিকে `unknown` টাইপ একই নমনীয়তা দেয়, কিন্তু safety ছাড়া না। এই পার্থক্যটা বোঝা একজন TypeScript developer-এর জন্য অত্যন্ত জরুরি।


## What is `any` — and Why is it a "Type Safety Hole"?

`any` টাইপ TypeScript-কে বলে:"এই ভেরিয়েবলের ব্যাপারে কোনো চেক করো না।" ফলে যেকোনো অপারেশন, যেকোনো প্রপার্টি অ্যাক্সেস — সব কিছু compile-time-এ বৈধ বলে মনে করা হয়।

```typescript
let data: any = "Hello";
data = 42;
data = { name: "Alice" };

// TypeScript কোনো error দেবে না — কিন্তু runtime-এ crash করতে পারে
console.log(data.toUpperCase()); // runtime error if data is a number
```

`any` ব্যবহার করলে আপনি essentially TypeScript বন্ধ করে JavaScript-এ ফিরে যাচ্ছেন। এটি একটি **type safety hole** কারণ:

- TypeScript কোনো ভুল ধরতে পারে না
- IDE auto-complete কাজ করে না
- Refactoring বিপজ্জনক হয়ে পড়ে
- Runtime errors production-এ গিয়ে দেখা দেয়

---

## What is `unknown` — The Safer Alternative

`unknown` টাইপও যেকোনো ভ্যালু ধরতে পারে, কিন্তু পার্থক্য হলো — **আপনি সরাসরি `unknown` ভেরিয়েবলের উপর কোনো অপারেশন করতে পারবেন না** যতক্ষণ না আপনি TypeScript-কে বলছেন এর actual type কী।

```typescript
let data: unknown = "Hello";

// Error! TypeScript অনুমতি দেবে না
// console.log(data.toUpperCase());

// সঠিক পদ্ধতি: type narrowing করতে হবে
if (typeof data === "string") {
  console.log(data.toUpperCase()); // এখন নিরাপদ
}
```

এটাই `unknown` কে শক্তিশালী করে — এটি আপনাকে **type check করতে বাধ্য করে** before use.

---

## Type Narrowing Explained

**Type Narrowing** হলো সেই প্রক্রিয়া যেখানে TypeScript একটি broad টাইপকে (যেমন `unknown` বা `string | number`) একটি specific টাইপে সংকুচিত করে — runtime check-এর ভিত্তিতে।

### Using `typeof` for Primitives

```typescript
type StringOrNumber = string | number;

function processValue(value: StringOrNumber): string {
  if (typeof value === "string") {
    return `String length: ${value.length}`; // TypeScript জানে এটা string
  }
  return `Number doubled: ${value * 2}`; // TypeScript জানে এটা number
}
```

### Using `instanceof` for Classes

```typescript
class Cat {
  meow() { return "Meow!"; }
}

class Dog {
  bark() { return "Woof!"; }
}

function makeSound(animal: Cat | Dog): string {
  if (animal instanceof Cat) {
    return animal.meow(); // narrowed to Cat 
  }
  return animal.bark(); // narrowed to Dog 
```

### Using Custom Type Guards

```typescript
interface Admin {
  role: "admin";
  permissions: string[];
}

interface User {
  role: "user";
  email: string;
}

function isAdmin(person: Admin | User): person is Admin {
  return person.role === "admin";
}

function handlePerson(person: Admin | User) {
  if (isAdmin(person)) {
    console.log(person.permissions); // Admin হিসেবে narrowed
  } else {
    console.log(person.email); // User হিসেবে narrowed
  }
}
```

---

## `any` vs `unknown` — Side by Side

| Feature | `any` | `unknown` |
| Accepts all values |Yes|Yes|
| Requires type check before use | No | Yes |
| Provides type safety | No | Yes |
| IDE autocomplete works | No | Yes after narrowing |
| Recommended for unpredictable data | No | Yes |

---

## Real-World Example: Handling API Response

```typescript
// Dangerous — any disables all checks
async function fetchDataBad(): Promise<any> {
  const response = await fetch("/api/user");
  const data = await response.json();
  return data.name.toUpperCase(); // No safety — might crash!
}

// Safe — unknown forces you to verify
async function fetchDataGood(): Promise<string> {
  const response = await fetch("/api/user");
  const data: unknown = await response.json();

  if (
    typeof data === "object" &&
    data !== null &&
    "name" in data &&
    typeof (data as { name: unknown }).name === "string"
  ) {
    return (data as { name: string }).name.toUpperCase(); // Safe
  }

  throw new Error("Unexpected data format");
}
```


## Conclusion

`any` টাইপ একটি quick fix মনে হলেও এটি আসলে TypeScript-এর সবচেয়ে বড় সুবিধাটাই কেড়ে নেয়। `unknown` টাইপ ব্যবহার করলে আপনি unpredictable data handle করার নমনীয়তা পান, কিন্তু type safety বজায় থাকে। আর **type narrowing** হলো সেই tool যা দিয়ে আপনি `unknown` কে নিরাপদে ব্যবহার করেন।

সংক্ষেপে: **বাইরে থেকে আসা যেকোনো data-এর জন্য `unknown` ব্যবহার করুন, `any` নয়।**