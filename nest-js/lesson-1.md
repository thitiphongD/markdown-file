---
title: Lession
slug: example-post
date: 2020-11-05 11:20:38
draft: false
layout: ../layouts/PostLayout.astro
description: Eos eu docendi tractatos sapientem, brute option menandri in vix, quando vivendo accommodare te ius. Nec melius fastidii constituam id, viderer theophrastus ad sit, hinc semper periculis cum id. Noluisse postulant assentior est in
---

# 📖 บทที่ 1: รู้จักและติดตั้ง NestJS

## 🎯 NestJS คืออะไร?

**NestJS** เป็น Node.js Framework ที่ใช้ TypeScript เป็นหลัก ได้แรงบันดาลใจมาจาก Angular โดยมีจุดเด่นคือ:

- 🏗️ **Architecture ที่เป็นระเบียบ** - ใช้ Module, Controller, Service pattern
- ✨ **Decorator ที่ทำให้โค้ดสะอาดและอ่านง่าย**
- 🔌 **Built-in Dependency Injection**
- 📝 **Support TypeScript เต็มรูปแบบ**
- 🛡️ **มาพร้อมฟีเจอร์ต่าง ๆ** เช่น Guards, Pipes, Interceptors

## 🚀 เริ่มสร้างโปรเจกต์แรกกัน!

### 📋 ขั้นตอนการติดตั้ง

```bash
# 1. ติดตั้ง NestJS CLI
npm install -g @nestjs/cli

# 2. สร้างโปรเจกต์ใหม่
nest new my-api-project

# 3. เข้าไปในโปรเจกต์
cd my-api-project

# 4. รันเซิร์ฟเวอร์
npm run start:dev
```

> **💡 ทิป**: เมื่อรันเสร็จ ไปที่ `http://localhost:3000` จะเห็น "Hello World!" ครับ

## 🏗️ ทำความเข้าใจโครงสร้างโปรเจกต์

### 📁 โครงสร้างไฟล์

```
my-api-project/
├── src/
│   ├── app.controller.ts      # จัดการ HTTP requests
│   ├── app.controller.spec.ts # ไฟล์ test
│   ├── app.module.ts          # Module หลักของแอพ
│   ├── app.service.ts         # Business logic
│   └── main.ts                # Entry point ของแอพ
├── package.json
├── tsconfig.json
└── nest-cli.json
```

## 📝 อธิบายไฟล์สำคัญ

### 🚪 `main.ts` - Entry Point

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

### 🧩 `app.module.ts` - Module หลัก

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [],                    // นำ modules อื่นมาใช้
  controllers: [AppController],   // ลงทะเบียน controllers
  providers: [AppService],        // ลงทะเบียน services
})
export class AppModule {}
```

## 🔧 มาดู Controller และ Service กันครับ

### 🎮 `app.controller.ts` - จัดการ HTTP Requests

```typescript
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller() // Decorator บอกว่านี่คือ Controller
export class AppController {
  // Dependency Injection - ฉีด AppService เข้ามา
  constructor(private readonly appService: AppService) {}

  @Get() // Decorator บอกว่านี่คือ GET route
  getHello(): string {
    return this.appService.getHello();
  }
}
```

### ⚙️ `app.service.ts` - Business Logic

```typescript
import { Injectable } from '@nestjs/common';

@Injectable() // Decorator บอกว่านี่คือ Service ที่สามารถ inject ได้
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

## 📚 สรุปบทที่ 1

### 🎯 สิ่งที่เราได้เรียนรู้

- ✅ **NestJS ใช้ Decorator pattern** และ Dependency Injection
- ✅ **โครงสร้าง**: Module → Controller → Service
- ✅ **Controller** จัดการ HTTP requests
- ✅ **Service** จัดการ business logic
- ✅ **Module** เป็นตัวรวม controllers และ services เข้าด้วยกัน

### 🔑 Key Concepts

| แนวคิด | คำอธิบาย |
|--------|----------|
| **Decorator** | ใช้ `@` prefix เพื่อเพิ่ม metadata ให้กับ class, method, property |
| **Dependency Injection** | ระบบที่ NestJS จัดการ dependencies ให้อัตโนมัติ |
| **Module Pattern** | การแบ่งแยกโค้ดเป็นส่วนๆ ที่ชัดเจนและสามารถนำกลับมาใช้ได้ |


## 🚀 พร้อมไปบทถัดไปแล้วมั้ยครับ?

เราจะเริ่มติดตั้ง **Prisma** และเชื่อมต่อกับฐานข้อมูลกันเลย! 

### 📋 สิ่งที่จะได้เรียนรู้ในบทถัดไป

- 🗄️ **Prisma คืออะไร** และทำงานอย่างไร
- 🔧 **การติดตั้งและ config** Prisma
- 🔗 **การเชื่อมต่อกับฐานข้อมูล** (เริ่มต้นด้วย SQLite)
- 📝 **การสร้าง Schema แรก**
