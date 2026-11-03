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
```bash
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
```bash

```
---

####
```bash

```
---
