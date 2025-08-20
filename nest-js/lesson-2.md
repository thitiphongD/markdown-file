# 📖 บทที่ 2: เริ่มต้นกับ Prisma

## 🎯 Prisma คืออะไร?

**Prisma** เป็น Database Toolkit ที่ทำให้การทำงานกับฐานข้อมูลง่ายขึ้น โดยมีจุดเด่น:

- 🛡️ **Type-safe** - ได้ TypeScript types อัตโนมัติ
- 💡 **Auto-completion** - IDE ช่วย suggest ได้
- 📝 **Schema-first** - เขียน schema แล้วได้ database + client
- 🔄 **Migration system** - จัดการเปลี่ยนแปลง database
- 🌐 **รองรับหลาย database** - PostgreSQL, MySQL, SQLite, MongoDB


## 🚀 เริ่มต้นกับ Prisma

### 📋 ขั้นตอนการติดตั้ง

```bash
# ใน folder my-api-project

# 1. ติดตั้ง Prisma
npm install prisma @prisma/client

# 2. เริ่มต้น Prisma (สร้างไฟล์ config)
npx prisma init --datasource-provider sqlite
```

### 📁 ไฟล์ที่จะถูกสร้าง

```
# ├── prisma/
# │   └── schema.prisma    # Schema ของฐานข้อมูล
# └── .env                 # Environment variables
```

## 📝 ดูไฟล์ที่ Prisma สร้างให้

### 🗄️ `prisma/schema.prisma`

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"           // ใช้ SQLite เพื่อความง่าย
  url      = env("DATABASE_URL") // อ่านจากไฟล์ .env
}

// ตรงนี้จะเพิ่ม models ในบทถัดไป
```

### 🔐 `.env`

```env
DATABASE_URL="file:./dev.db"
```

## 🏗️ สร้าง Model แรกกัน!

มาสร้าง Model สำหรับจัดการผู้ใช้ (User) กันครับ:

### 👤 Model User - ตารางผู้ใช้

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

// Model User - ตารางผู้ใช้
model User {
  id        Int      @id @default(autoincrement()) // Primary key, auto increment
  email     String   @unique                       // ห้ามซ้ำ
  name      String?                                // สามารถเป็น null ได้
  createdAt DateTime @default(now())               // วันที่สร้าง (ค่าเริ่มต้นคือตอนนี้)
  updatedAt DateTime @updatedAt                    // วันที่อัพเดท (อัตโนมัติ)
}
```

## 📚 อธิบาย Prisma Schema

### 🔑 Field Decorators ที่สำคัญ

ใน schema ด้านบน:

| Decorator | ความหมาย |
|-----------|----------|
| `@id` | **Primary key** - คีย์หลักของตาราง |
| `@default(autoincrement())` | เลข id เพิ่มอัตโนมัติ |
| `@unique` | **ห้ามซ้ำ** - ค่าต้องไม่ซ้ำกับแถวอื่น |
| `String?` | สามารถเป็น **null** ได้ (optional) |
| `@default(now())` | ค่าเริ่มต้นคือเวลาปัจจุบัน |
| `@updatedAt` | อัพเดทเวลาอัตโนมัติเมื่อมีการแก้ไข |


## 🔄 รัน Migration และ Generate Client

### 📋 ขั้นตอนการ Migration

```bash
# 1. สร้าง migration (เปลี่ยนแปลงฐานข้อมูล)
npx prisma migrate dev --name init

# คำสั่งนี้จะ:
# - สร้างไฟล์ SQL migration
# - สร้างฐานข้อมูล SQLite (dev.db)
# - Generate Prisma Client

# 2. ถ้าอยากดู database ผ่าน GUI
npx prisma studio
```

---

## 🧪 ทดสอบการเชื่อมต่อ

### 🔧 สร้างไฟล์ทดสอบ

#### 📝 `src/prisma.service.ts` - สร้างไฟล์ใหม่

```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  async onModuleInit() {
    await this.$connect();
    console.log('✅ Connected to database successfully!');
  }
}
```

#### 🧩 อัปเดต `app.module.ts` - เพิ่ม PrismaService

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { PrismaService } from './prisma.service';

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService, PrismaService], // เพิ่มตรงนี้
})
export class AppModule {}
```

## 🎯 ทดสอบสร้าง User แรก

### 🔧 แก้ไข `app.controller.ts` - เพิ่ม route ทดสอบ

```typescript
import { Controller, Get, Post } from '@nestjs/common';
import { AppService } from './app.service';
import { PrismaService } from './prisma.service';

@Controller()
export class AppController {
  constructor(
    private readonly appService: AppService,
    private readonly prisma: PrismaService, // Inject PrismaService
  ) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }

  // เพิ่ม route ทดสอบสร้าง user
  @Post('test-user')
  async createTestUser() {
    const user = await this.prisma.user.create({
      data: {
        email: 'test@example.com',
        name: 'Test User',
      },
    });
    return { message: 'User created!', user };
  }

  // ดู users ทั้งหมด
  @Get('users')
  async getUsers() {
    return this.prisma.user.findMany();
  }
}
```

## 🚀 ทดสอบการทำงาน

### 📋 ขั้นตอนการทดสอบ

```bash
# รัน server
npm run start:dev
```

### 🧪 ทดสอบด้วย

| Method | Endpoint | หน้าที่ |
|--------|----------|---------|
| `POST` | `http://localhost:3000/test-user` | สร้าง user |
| `GET` | `http://localhost:3000/users` | ดู users ทั้งหมด |


## 📚 สรุปบทที่ 2

### ✅ สิ่งที่เราได้เรียนรู้

- ✅ **ติดตั้ง Prisma** และเชื่อมต่อกับ SQLite
- ✅ **สร้าง Schema** ตาราง User ด้วย Prisma Schema Language
- ✅ **รัน Migration** เพื่อสร้างฐานข้อมูล
- ✅ **Generate Prisma Client** ที่มี type-safety
- ✅ **ทดสอบ CRUD พื้นฐาน**

### 🔑 Key Concepts

| แนวคิด | คำอธิบาย |
|--------|----------|
| **Prisma Schema** | ไฟล์ที่กำหนดโครงสร้างฐานข้อมูลและ models |
| **Migration** | การเปลี่ยนแปลงโครงสร้างฐานข้อมูลแบบ version control |
| **Prisma Client** | TypeScript client ที่สร้างจาก schema เพื่อใช้งานฐานข้อมูล |


## 🚀 ในบทถัดไป

เราจะเรียนรู้ **Prisma Schema อย่างละเอียด** การสร้าง **Relations ระหว่างตาราง** และ **Advanced Features** ต่าง ๆ!

### 📋 สิ่งที่จะได้เรียนรู้

- 🔗 **Relationships** ระหว่างตาราง (One-to-One, One-to-Many, Many-to-Many)
- 📝 **Advanced Schema** features
- 🔍 **Complex Queries** และ filtering
- 🛡️ **Validation** และ error handling
