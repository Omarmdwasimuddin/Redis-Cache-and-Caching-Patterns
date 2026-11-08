# NestJS CacheModule + Redis Configuration

> NestJS-এর **CacheModule**-এর সঙ্গে **Redis** সংযোগ করে Redis-কে **Cache Storage** হিসেবে ব্যবহার করার Configuration।

---

## 📌 সংক্ষেপে (TL;DR)

```
NestJS App ──► CacheModule ──► redisStore ──► Redis Server (localhost:6379)
                                   TTL: 60 seconds
```

| Option | কাজ |
|---|---|
| `CacheModule` | NestJS-এ Caching সুবিধা দেয় |
| `redisStore` | Redis-কে Cache Storage হিসেবে ব্যবহার করে |
| `registerAsync()` | Asynchronous Configuration করে |
| `isGlobal` | পুরো Application-এ CacheModule Available করে |
| `useFactory` | Configuration Object তৈরি করে |
| `socket` | Redis Connection Information দেয় |
| `ttl` | Cache কতক্ষণ থাকবে তা নির্ধারণ করে |

---

## 📑 সূচিপত্র

1. [পুরো কোড](#1-পুরো-কোড)
2. [CacheModule কী?](#2-cachemodule-কী)
3. [redisStore কী?](#3-redisstore-কী)
4. [registerAsync()](#4-registerasync)
5. [isGlobal: true](#5-isglobal-true)
6. [useFactory](#6-usefactory)
7. [store](#7-store)
8. [socket](#8-socket)
9. [ttl](#9-ttl)
10. [সম্পূর্ণ Flow](#10-সম্পূর্ণ-flow)
11. [CACHE_MANAGER ব্যবহার](#11-cache_manager-ব্যবহার)
12. [Quick Revision](#12-quick-revision)

---

## 1. পুরো কোড

```ts
import { Module } from '@nestjs/common';
import { CacheModule } from '@nestjs/cache-manager';
import { redisStore } from 'cache-manager-redis-store';

@Module({
  imports: [
    CacheModule.registerAsync({
      isGlobal: true,
      useFactory: async () => ({
        store: await redisStore({
          socket: {
            host: 'localhost',
            port: 6379,
          },
          ttl: 60, // seconds
        }),
      }),
    }),
  ],
})
export class AppModule {}
```

এবার প্রতিটি অংশ বুঝি 👇

---

## 2. `CacheModule` কী?

```ts
import { CacheModule } from '@nestjs/cache-manager';
```

`CacheModule` হলো NestJS-এর **built-in caching integration module**। এটি Application-এ **Cache Management**-এর সুবিধা দেয়।

প্রতিবার একই data-র জন্য Database Query করলে অতিরিক্ত সময় লাগে। Cache ব্যবহার করে data সাময়িকভাবে রেখে দেওয়া যায়।

```ts
@Get('products')
getProducts() {
  return this.appService.getProducts();
}
```

প্রতিবার API Call-এ Database থেকে Products না এনে **Cache থেকে** data পাওয়া সম্ভব।

---

## 3. `redisStore` কী?

```ts
import { redisStore } from 'cache-manager-redis-store';
```

`redisStore` হলো Redis-এর সঙ্গে Cache Manager-এর সংযোগের জন্য **Store Adapter**।

| কে | কাজ |
|---|---|
| `CacheModule` | Caching System দেয় |
| `redisStore` | Cache data **কোথায় Store হবে** তা বলে দেয় |
| Redis | সেই data **Store করে রাখে** |

CacheModule-এর Default Storage হিসেবে **Memory** ব্যবহার করা যায়। কিন্তু আমরা **Redis** চাই, তাই `redisStore` Import করেছি।

---

## 4. `CacheModule.registerAsync()`

```ts
imports: [
  CacheModule.registerAsync({ ... })
]
```

NestJS-এ Module দুইভাবে Configure করা যায়:

| Method | কাজ |
|---|---|
| `register()` | **Synchronous** Configuration |
| `registerAsync()` | **Asynchronous** Configuration |

আমরা `registerAsync()` ব্যবহার করেছি কারণ Redis Store তৈরির সময় **`await`** ব্যবহার করছি। অর্থাৎ Redis Store প্রস্তুত হওয়ার **পরে** CacheModule Configuration সম্পন্ন হবে।

---

## 5. `isGlobal: true`

```ts
isGlobal: true,
```

CacheModule-কে পুরো Application-এর জন্য **Global Module** হিসেবে Register করে।

সাধারণত অন্য Module-এ কোনো Module ব্যবহার করতে চাইলে সেটি **Import** করতে হয়। কিন্তু `isGlobal: true` থাকলে **আলাদাভাবে Import করতে হয় না**।

```ts
@Module({
  controllers: [ProductController],
  providers: [ProductService],
})
export class ProductModule {}
```

এখানে CacheModule Import না করেও Dependency Injection-এর মাধ্যমে Cache ব্যবহার করা যাবে।

---

## 6. `useFactory`

```ts
useFactory: async () => ({ ... })
```

`useFactory` হলো NestJS-এর **Dynamic Configuration** পদ্ধতি। একটি **Factory Function** তৈরি করা হয়, যা Module-এর Configuration Object **return** করে।

| অংশ | মানে |
|---|---|
| `async` | Function Asynchronous, ভেতরে `await` ব্যবহার করা যাবে |
| `() => ({ ... })` | Configuration Object return করবে |

---

## 7. `store`

```ts
store: await redisStore({ ... }),
```

`store` ঠিক করে Cache data **কোথায় এবং কীভাবে** Store হবে। এখানে **Redis Store** ব্যবহার হচ্ছে।

`await` কারণ Store Initialization **Asynchronous**।

---

## 8. `socket`

```ts
socket: {
  host: 'localhost',
  port: 6379,
},
```

Redis Server-এর **Connection Information**।

| Option | মানে |
|---|---|
| `host: 'localhost'` | Redis Server **Local Machine**-এ চলছে |
| `port: 6379` | Redis-এর **Default Port** |

> 🌐 **Production-এ** Redis অন্য Server-এ থাকলে সেই Server-এর **Hostname বা IP Address** দিতে হবে।

অর্থাৎ NestJS Application `localhost:6379` Address দিয়ে Redis-এর সঙ্গে Connect করবে।

---

## 9. `ttl`

```ts
ttl: 60, // seconds
```

**TTL = Time To Live।** Cache data কতক্ষণ সংরক্ষিত থাকবে তা নির্ধারণ করে। এখানে **৬০ সেকেন্ড**।

### TTL = 60 seconds — Visual Example

```
0 sec        20 sec        60 sec        Next Request
  │            │             │               │
  ▼            ▼             ▼               ▼
Cache MISS   Cache HIT    EXPIRED        Cache MISS
  │            │             │               │
Database      Redis থেকে    Cached data    আবার Database
থেকে data     Cached data   মুছে গেছে      থেকে data নিয়ে
নিয়ে Redis-এ পাওয়া গেল                    Cache করা হবে
Store করা হলো
```

**ধাপে ধাপে:**

1. প্রথমবার `/products` Call → Database থেকে data নিয়ে **Redis-এ Cache**
2. পরবর্তী ৬০ সেকেন্ডে একই Cache Key পাওয়া গেলে → **Cache থেকে data**
3. TTL শেষ → Cache Entry **Expire**
4. পরের Request-এ **Cache Miss** → আবার Database থেকে নিয়ে Cache

> ⚠️ **Version সতর্কতা:** TTL-এর unit (সেকেন্ড না মিলিসেকেন্ড) `cache-manager` ও store package-এর version-এর ওপর নির্ভর করতে পারে। তাই নিজের project-এর version অনুযায়ী Documentation চেক করে নিও এবং বাস্তবে TTL ঠিকমতো কাজ করছে কিনা পরীক্ষা করো।

---

## 10. সম্পূর্ণ Flow

```
┌──────────────────────┐
│ NestJS Application   │
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ AppModule            │
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ CacheModule          │
│ registerAsync()      │
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ redisStore           │
│ (Store Adapter)      │
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ Redis Server         │
│ localhost:6379       │
│ TTL: 60 Seconds      │
└──────────────────────┘
```

---

## 11. `CACHE_MANAGER` ব্যবহার

উদাহরণ Products data:

```ts
export const products = [
  { id: 1101, name: 'Macbook Pro', price: 25000 },
  { id: 1102, name: 'iPhone 17', price: 17000 },
  { id: 1103, name: 'iPad Pro', price: 20000 },
];
```

এই data Redis-এ Cache করতে `CACHE_MANAGER` ব্যবহার করা যায়:

```ts
import { CACHE_MANAGER } from '@nestjs/cache-manager';
import { Cache } from 'cache-manager';
import { Inject } from '@nestjs/common';

constructor(
  @Inject(CACHE_MANAGER)
  private cacheManager: Cache,
) {}
```

| Code | মানে |
|---|---|
| `CACHE_MANAGER` | Cache Manager-এর **Injection Token** |
| `@Inject()` | Cache Manager-কে **Dependency Injection** করে |
| `cacheManager` | Cache-এর data **Read, Write, Delete** করে |

---

## 12. Quick Revision

| বিষয় | কাজ |
|---|---|
| `CacheModule` | NestJS-এ Caching সুবিধা দেয় |
| `redisStore` | Redis-কে Cache Storage হিসেবে ব্যবহার করতে সাহায্য করে |
| `registerAsync()` | Asynchronous Configuration করে |
| `isGlobal` | পুরো Application-এ CacheModule Available করে |
| `useFactory` | Configuration Object তৈরি করে |
| `socket` | Redis Connection Information দেয় |
| `ttl` | Cache কতক্ষণ থাকবে তা নির্ধারণ করে |

---

## 🚀 পরবর্তী ধাপ

`CACHE_MANAGER` দিয়ে একটি সম্পূর্ণ **Cache-Aside Pattern** Implement করা:

```
cacheManager.get()   →  Cache থেকে data পড়া
cacheManager.set()   →  Cache-এ data রাখা
cacheManager.del()   →  Cache থেকে data মুছে ফেলা
```
