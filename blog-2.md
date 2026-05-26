# How `Pick` and `Omit` Keep Your TypeScript Code DRY

## Introduction

বড় TypeScript project-এ একটি সাধারণ সমস্যা হলো — একই interface বা তার অংশবিশেষ বারবার লেখা। এতে code duplication বাড়ে, এবং একটা জায়গায় পরিবর্তন করলে আরও অনেক জায়গায় করতে হয়। TypeScript-এর `Pick` এবং `Omit` utility types এই সমস্যার elegant সমাধান দেয় এবং কোডকে **DRY (Don't Repeat Yourself)** রাখতে সাহায্য করে।


## The Problem: Code Duplication Without Utility Types

ধরুন আপনার একটি master interface আছে:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  role: "admin" | "user";
  createdAt: Date;
}
```

এখন বিভিন্ন জায়গায় আপনার User-এর ভিন্ন ভিন্ন "slice" দরকার:

```typescript
// Without utility types — code duplication
interface UserProfile {
  id: number;
  name: string;
  email: string;
}

interface UserCard {
  name: string;
  role: "admin" | "user";
}

interface UserWithoutPassword {
  id: number;
  name: string;
  email: string;
  role: "admin" | "user";
  createdAt: Date;
}
```

যদি `User` interface-এ কোনো পরিবর্তন আসে, তাহলে এই তিনটি interface-ও আলাদা আলাদা করে আপডেট করতে হবে। এটাই DRY violation।

---

## `Pick` — Select Only What You Need

`Pick<Type, Keys>` একটি type তৈরি করে যেখানে শুধু আপনার নির্বাচিত properties থাকে।

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  role: "admin" | "user";
  createdAt: Date;
}

// শুধু দরকারি fields নিন
type UserProfile = Pick<User, "id" | "name" | "email">;
type UserCard = Pick<User, "name" | "role">;

// এখন UserProfile এরকম দেখায়:
// { id: number; name: string; email: string; }
```

### Practical Example with `Pick`

```typescript
// API response-এ শুধু public info পাঠানো
function getPublicProfile(user: User): UserProfile {
  return {
    id: user.id,
    name: user.name,
    email: user.email,
  };
}

// UI card-এর জন্য minimal data
function renderUserCard(user: User): UserCard {
  return {
    name: user.name,
    role: user.role,
  };
}
```

## `Omit` — Exclude What You Don't Want

`Omit<Type, Keys>` `Pick`-এর বিপরীত — এটি নির্দিষ্ট properties বাদ দিয়ে বাকি সব রাখে।

```typescript
// password বাদ দিয়ে বাকি সব রাখুন
type SafeUser = Omit<User, "password">;

// এখন SafeUser এরকম দেখায়:
// { id: number; name: string; email: string; role: "admin" | "user"; createdAt: Date; }

// একাধিক field বাদ দেওয়া
type PublicUser = Omit<User, "password" | "createdAt">;
```

### Practical Example with `Omit`

```typescript
// Database থেকে user আনার পর password ছাড়া পাঠানো
function sanitizeUser(user: User): SafeUser {
  const { password, ...safeUser } = user;
  return safeUser;
}

// New user তৈরির form — id এবং createdAt এখনো নেই
type CreateUserInput = Omit<User, "id" | "createdAt">;

function createUser(input: CreateUserInput): User {
  return {
    ...input,
    id: Math.random(),
    createdAt: new Date(),
  };
}
```

## Combining `Pick` and `Omit` for Complex Scenarios

```typescript
interface Product {
  id: number;
  name: string;
  price: number;
  stock: number;
  supplierId: number;
  internalNotes: string;
}

// Customer দেখবে — supplier ও internal info বাদ
type PublicProduct = Omit<Product, "supplierId" | "internalNotes">;

// Admin dashboard-এ শুধু inventory info
type InventoryView = Pick<Product, "id" | "name" | "stock">;

// নতুন product তৈরিতে — id নেই
type NewProduct = Omit<Product, "id">;
```


## Why This Keeps Code DRY

| Without Utility Types | With `Pick` / `Omit` |
| প্রতিটি "slice"-এর জন্য আলাদা interface | Master interface থেকে derive করা |
| Master পরিবর্তনে সব interface manually আপডেট | Master পরিবর্তনে সব derived type auto-update |
| Repetition এবং inconsistency-র ঝুঁকি | Single source of truth |
| বড় codebase-এ maintain করা কঠিন | Refactoring সহজ এবং নিরাপদ |


## Conclusion

`Pick` এবং `Omit` হলো TypeScript-এর অন্যতম শক্তিশালী utility types। এগুলো আপনাকে একটি **master interface** থেকে বিভিন্ন purpose-এর জন্য **specialized "slices"** তৈরি করতে দেয় — কোনো duplication ছাড়াই। যখনই আপনি দেখবেন একই interface-এর properties বিভিন্ন জায়গায় copy করছেন, সেটাই `Pick` বা `Omit` ব্যবহার করার সংকেত।

*** একটাই সত্য (Single Source of Truth) — master interface। বাকি সব তার থেকে derive করুন।