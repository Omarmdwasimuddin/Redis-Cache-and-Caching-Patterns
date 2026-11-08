## Redis-Cache-and-Caching-Patterns

####
```bash
wsl -d Ubuntu
```
---

####
```bash
sudo apt install redis-server -y
```
---

####
```bash
sudo systemctl enable redis-server
```
---

####
```bash
sudo systemctl start redis-server
```
---

####
```bash
sudo systemctl status redis-server
```
```bash
Ctrl+c
```
---

####
```bash
redis-cli ping
```
---

####
```bash
npx create-nx-workspace@latest wasutech-cache
```
```bash
cd wasutech-cache
```
---

####
```bash
npm i @nx/nest
```
---

####
```bash
npx nx g @nx/nest:app product-service
```
---

####
```bash
npm i redis @nestjs/cache-manager cache-manager cache-manager-redis-store
```
```bash
code .
```
---

#### `product-service/src/app/fake-db.ts`
```ts
export const products = [
    {
        id: 1101,
        name: 'Macbook Pro',
        price: 25000,
    },
    {
        id: 1102,
        name: 'iPhone 17',
        price: 17000,
    },
    {
        id: 1103,
        name: 'iPad Pro',
        price: 20000,
    }
]
```
---

#### `product-service/src/app/app.module.ts`
```ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { CacheModule } from '@nestjs/cache-manager';
import { redisStore } from 'cache-manager-redis-store';


@Module({
  imports: [
    CacheModule.registerAsync({
      isGlobal: true,
      useFactory: async() => ({
        store: await redisStore({
          socket: {
            host: 'localhost',
            port: 6379,
          },
          ttl: 60, // seconds
        }),
      })
    })
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```
---

#### `product-service/src/app/app.service.ts`
```ts
import { Injectable, Inject } from '@nestjs/common';
import { CACHE_MANAGER } from '@nestjs/cache-manager';
import type { Cache } from 'cache-manager';
import { products } from './fake-db';

@Injectable()
export class AppService {
  constructor(
    @Inject(CACHE_MANAGER) private readonly cacheManager: Cache,
  ) {}

  async getProducts(id: number) {
    const cacheKey = `product:${id}`;
    const cachedProduct = await this.cacheManager.get(cacheKey);

    if (cachedProduct) {
      console.log('Cache Hit')
      return cachedProduct;
    }
    console.log('Cache Miss')

    const product = products.find((p) => p.id === id);
    await this.cacheManager.set(
      cacheKey,
      product,
    );
    return product;
  }

  async updateProduct(
    id: number,
    price: number,
  ) {
    const product = products.find((p) => p.id === id)
    if(!product) {
      return {
        message: 'Product not found!',
      }
    }
    product.price = price;
    await this.cacheManager.set(
      `product:${id}`,
      product,
    );
    console.log('Cache Updated')
      return {
        message: 'Product update successfully!',
        product
      }
    
  }

}
```
---

#### `product-service/src/app/app.controller.ts`
```ts
import { Controller, Get, Param, Patch } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get('products/:id')
  async getProduct(@Param('id') id: string) {
    return this.appService.getProducts(Number(id));
  }

  @Patch('products/:id/:price')
  async updateProduct(
    @Param('id') id: string,
    @Param('price') price: string,
  ) {
    return this.appService.updateProduct(Number(id), Number(price));
  }
}
```
---

#### run koro
```bash
npx nx serve product-service
```
```bash
http://localhost:3000/api/products/1101
```
<img width="583" height="199" alt="image" src="https://github.com/user-attachments/assets/d998268b-24e8-4c8c-b43b-7c0a9250c138" />

---

#### Extention theke Thunder Client install koro
<img width="1344" height="463" alt="image" src="https://github.com/user-attachments/assets/25815641-8b47-4c7a-9827-cd671b4e9a78" />

---

####
