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
---

####
```bash

```
---
