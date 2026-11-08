# NestJS + Redis: Cache-Aside Implementation (Product Read ও Update)

> `CACHE_MANAGER` ব্যবহার করে Redis Cache থেকে **Product Read** এবং **Update** করা। এখানে মূল technique হলো **Cache-Aside Pattern (Lazy Loading)**।

---

## 📌 সংক্ষেপে (TL;DR)

```
getProducts(id):   Redis আছে? ──Yes──► Return (Cache Hit)
                        │
                        No (Cache Miss)
                        ▼
                  Data Source থেকে আনো ──► Redis-এ Store ──► Return

updateProduct(id): Data Update ──► Redis-এ নতুন data Set ──► Return
```

| Method | কাজ |
|---|---|
| `cacheManager.get(key)` | Cache থেকে Data **Read** |
| `cacheManager.set(key, value)` | Cache-এ Data **Store** |
| `cacheManager.del(key)` | Cache থেকে Data **Delete** |
| `cacheManager.clear()` | Cache Store-এর সব Data **Clear** |

---

## 📑 সূচিপত্র

1. [পুরো কোড](#1-পুরো-কোড)
2. [CACHE_MANAGER কী?](#2-cache_manager-কী)
3. [Cache Type](#3-cache-type)
4. [Constructor ও Dependency Injection](#4-constructor-ও-dependency-injection)
5. [getProducts()](#5-getproducts)
6. [getProducts() Flow](#6-getproducts-flow)
7. [updateProduct()](#7-updateproduct)
8. [দুটি Method একসঙ্গে](#8-দুটি-method-একসঙ্গে)
9. [কোড Improve করা](#9-কোড-improve-করা)
10. [নিজের জ্ঞান যাচাই করো](#10-নিজের-জ্ঞান-যাচাই-করো)

---

## 1. পুরো কোড

```ts
import { Inject, Injectable } from '@nestjs/common';
import { CACHE_MANAGER } from '@nestjs/cache-manager';
import type { Cache } from 'cache-manager';

export const products = [
  { id: 1101, name: 'Macbook Pro', price: 25000 },
  { id: 1102, name: 'iPhone 17', price: 17000 },
  { id: 1103, name: 'iPad Pro', price: 20000 },
];

@Injectable()
export class AppService {
  constructor(
    @Inject(CACHE_MANAGER) private readonly cacheManager: Cache,
  ) {}

  // Product Read (Cache-Aside)
  async getProducts(id: number) {
    const cacheKey = `product:${id}`;

    const cachedProduct = await this.cacheManager.get(cacheKey);

    if (cachedProduct) {
      console.log('Cache Hit');
      return cachedProduct;
    }

    console.log('Cache Miss');

    const product = products.find((p) => p.id === id);

    await this.cacheManager.set(cacheKey, product);

    return product;
  }

  // Product Update + Cache Update
  async updateProduct(id: number, price: number) {
    const product = products.find((p) => p.id === id);

    if (!product) {
      return { message: 'Product not found!' };
    }

    product.price = price;

    await this.cacheManager.set(`product:${id}`, product);

    console.log('Cache Updated');

    return {
      message: 'Product update successfully!',
      product,
    };
  }
}
```

এবার এক এক করে বুঝি 👇

---

## 2. `CACHE_MANAGER` কী?

```ts
import { CACHE_MANAGER } from '@nestjs/cache-manager';
```

`CACHE_MANAGER` হলো NestJS Cache Module-এর একটি **Injection Token**। এর মাধ্যমে **Dependency Injection** ব্যবহার করে Cache Manager-কে Service বা Controller-এ আনা যায়।

`AppModule`-এ `CacheModule` Configure করলে NestJS একটি Cache Manager তৈরি করে। সেটাকে `AppService`-এ ব্যবহার করতেই `CACHE_MANAGER` Import করা হয়েছে।

```ts
@Inject(CACHE_MANAGER)
private readonly cacheManager: Cache
```

অর্থ: NestJS-কে বলা হচ্ছে — "আমাকে Cache Manager-এর Instance Inject করে দাও।"

---

## 3. `Cache` Type

```ts
import type { Cache } from 'cache-manager';
```

`Cache` হলো একটি **TypeScript Type**, যা `cacheManager` Object-এর Type নির্ধারণ করে। এর ফলে TypeScript জানে `cacheManager`-এ কী কী method আছে (`get`, `set`, `del`, `clear`)।

> 💡 **`import type` কেন?** `Cache` শুধু **Type Checking**-এর জন্য দরকার। **Runtime-এ** এই Import-এর প্রয়োজন নেই।

---

## 4. Constructor ও Dependency Injection

```ts
constructor(
  @Inject(CACHE_MANAGER) private readonly cacheManager: Cache,
) {}
```

| অংশ | মানে |
|---|---|
| `@Inject(CACHE_MANAGER)` | `CACHE_MANAGER` Token-এ Registered Dependency এখানে Inject করো |
| `private` | শুধু এই Class-এর ভেতরে ব্যবহার করা যাবে |
| `readonly` | Constructor-এ Assign হওয়ার পর বদলানো যাবে না |
| `cacheManager` | Variable-এর নাম |
| `Cache` | Variable-এর Type |

NestJS নিজেই Cache Manager-এর Instance তৈরি করে Constructor-এর মাধ্যমে দেয়। তাই নিজে Object তৈরি করতে হয় না।

---

## 5. `getProducts()` — Cache থেকে Product Read

```ts
async getProducts(id: number) {
```

একটি **Asynchronous Method**, যা `id` (number) নেয়। যেমন `getProducts(1101)`।

### Step 1: Cache Key তৈরি

```ts
const cacheKey = `product:${id}`;
```

`id = 1101` হলে `cacheKey = 'product:1101'`।

Redis-এ প্রতিটি data একটি **Key**-এর বিপরীতে Store হয়:

| Key | Value |
|---|---|
| `product:1101` | Macbook Pro |
| `product:1102` | iPhone 17 |
| `product:1103` | iPad Pro |

`product:` হলো **Prefix**, `id` হলো **Unique Identifier**। এভাবে বিভিন্ন Product-এর Cache আলাদা করে Manage করা সহজ হয়।

### Step 2: Cache থেকে Data Read

```ts
const cachedProduct = await this.cacheManager.get(cacheKey);
```

Redis-কে জিজ্ঞেস করা হচ্ছে: `product:1101` Key-এর বিপরীতে কোনো data আছে কি?

| অবস্থা | ফলাফল |
|---|---|
| Data আছে | Product Return করে |
| Data নেই | `undefined` Return করে |

### Step 3: Cache Hit ✅

```ts
if (cachedProduct) {
  console.log('Cache Hit');
  return cachedProduct;
}
```

Data পাওয়া গেলে **Cache Hit**। Redis থেকে পাওয়া data সরাসরি Return হয়, অন্য Data Source থেকে নতুন করে আনতে হয় না।

### Step 4: Cache Miss ❌

```ts
console.log('Cache Miss');
```

Data না পেলে `if` Block চলে না, তখন **Cache Miss** Log হয়। এখন অন্য Data Source থেকে data আনতে হবে।

### Step 5: Products Array থেকে খোঁজা

```ts
const product = products.find((p) => p.id === id);
```

| অংশ | মানে |
|---|---|
| `p` | Array-এর প্রতিটি Product |
| `p.id` | Product-এর ID |
| `===` | ID দুটি সমান কিনা চেক |

পেলে Product Return করে, না পেলে `undefined`।

### Step 6: Redis-এ Store

```ts
await this.cacheManager.set(cacheKey, product);
```

Syntax: `cacheManager.set(key, value)`

```
Key:   product:1101
Value: { id: 1101, name: 'Macbook Pro', price: 25000 }
```

`await` থাকায় Store শেষ হওয়ার পর পরের লাইন চলে।

### Step 7: Product Return

```ts
return product;
```

প্রথম Request-এ Cache Miss হলেও **পরের Request-এর জন্য** data Redis-এ থেকে যায়।

---

## 6. `getProducts()` Flow

```
Client Request: GET /products/1101
        │
        ▼
Generate Cache Key: product:1101
        │
        ▼
Redis Cache Check: cacheManager.get(cacheKey)
        │
   ┌────┴─────────────────────┐
   ▼                          ▼
Cache HIT ✅              Cache MISS ❌
Data Found                Data Not Found
   │                          │
   ▼                          ▼
Redis থেকে Return        Products Array থেকে Search
                              │
                              ▼
                          Redis-এ Store
                              │
                              ▼
                          Data Return
```

---

## 7. `updateProduct()` — Product ও Cache Update

```ts
async updateProduct(id: number, price: number) {
```

| Parameter | মানে |
|---|---|
| `id` | কোন Product Update হবে |
| `price` | নতুন Price |

উদাহরণ: `updateProduct(1101, 30000)` → ID `1101`-এর Price হবে `30000`।

### Step 1: Product খোঁজা

```ts
const product = products.find((p) => p.id === id);
```

### Step 2: Product পাওয়া গেছে কিনা চেক

```ts
if (!product) {
  return { message: 'Product not found!' };
}
```

`!` হলো **Logical NOT**। Product না পেলে `product = undefined`, তাই `!product` হয় `true` এবং Error Message Return হয়। এটাকে **Early Return** বলে।

### Step 3: Price Update

```ts
product.price = price;
```

```
আগে:  { id: 1101, name: 'Macbook Pro', price: 25000 }
পরে:  { id: 1101, name: 'Macbook Pro', price: 30000 }
```

> 📝 `find()` থেকে পাওয়া Object মূল Array-এর **একই Object Reference**। তাই পরিবর্তনটি মূল `products` Array-তেও কার্যকর হয়।

### Step 4: Redis Cache Update

```ts
await this.cacheManager.set(`product:${id}`, product);
```

একই Key-এর বিপরীতে **নতুন Value Set** হয়, ফলে আগের Value বদলে যায়।

```
Key: product:1101
আগের Value: { ..., price: 25000 }
নতুন Value:  { ..., price: 30000 }
```

### Step 5 ও 6: Log ও Response

```ts
console.log('Cache Updated');

return {
  message: 'Product update successfully!',
  product,
};
```

Client-কে Success Message এবং Updated Product Return করা হয়।

> 🔎 **লক্ষ করো:** `getProducts()` **Cache-Aside** (Read-এর সময় Cache ভরে), আর `updateProduct()`-এ Write-এর সময় Cache-ও Update হচ্ছে, যা **Write-Through** ধরনের আচরণ।

---

## 8. দুটি Method একসঙ্গে

| # | Request | ফলাফল | কী ঘটে |
|---|---|---|---|
| ১ | `GET /products/1101` | ❌ Cache MISS | Products Array → Redis Cache |
| ২ | `GET /products/1101` | ✅ Cache HIT | Redis Cache → Client |
| ৩ | `PATCH /products/1101` `{ "price": 30000 }` | 🔄 Cache Updated | Products Array ও Redis দুটোই Updated |
| ৪ | `GET /products/1101` | ✅ Cache HIT | Redis → **নতুন Price 30000** |

---

## 9. কোড Improve করা

Learning Example হিসেবে কোড ভালো। তবে **Production Level**-এ দুটি বিষয় খেয়াল রাখতে হবে।

### ১) Product না পেলে `undefined` Cache করা উচিত নয়

বর্তমানে Product না পেলেও `set(cacheKey, undefined)` চলে। ঠিক করা:

```ts
const product = products.find((p) => p.id === id);

if (!product) {
  return null;
}

await this.cacheManager.set(cacheKey, product);

return product;
```

### ২) Cache Update ≠ Database Update

বর্তমান Example-এ `products` একটি **In-memory Array**। Production-এ:

```
Database Update ──► তারপর Redis Cache Update অথবা Invalidate
```

---

## 10. নিজের জ্ঞান যাচাই করো

**১. প্রথমবার কোনো Product Request করলে Redis-এ Data না থাকলে সেটিকে কী বলা হয়?**

- [ ] Cache Hit
- [ ] Cache Miss
- [ ] Cache Expired

**২. `cacheManager.set(cacheKey, product)` কী কাজ করে?**

- [ ] Cache থেকে Data Delete করে
- [ ] Cache থেকে Data Read করে
- [ ] Cache-এ Data Store করে

**৩. একই Product দ্বিতীয়বার Request করলে Cache-এ Data থাকলে কী হবে?**

- [ ] Cache Hit
- [ ] Cache Miss
- [ ] Database Delete

<details>
<summary>✅ উত্তর যাচাই করো</summary>

1. **Cache Miss**
2. **Cache-এ Data Store করে**
3. **Cache Hit**

</details>

---

## 🚀 পরবর্তী ধাপ

এই তিনটির **পার্থক্য** ভালোভাবে বোঝা:

- **Cache-Aside Pattern**
- **Write-Through Pattern**
- **Cache Invalidation**
