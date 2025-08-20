# 📖 บทที่ 3: ออกแบบ Database Schema

## 🎯 เรียนรู้ Prisma Schema Language

มาเรียนรู้ Prisma Schema แบบละเอียดกันครับ เราจะสร้าง Schema สำหรับระบบ Blog ที่มี Users, Posts, และ Categories!

### 🏗️ Blog System Schema

```prisma
// prisma/schema.prisma - Blog System Schema
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

// โมเดล User - ผู้ใช้
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  // Relations - ความสัมพันธ์
  posts     Post[]   // User หนึ่งคนสามารถมีหลาย Posts (One-to-Many)
}

// โมเดล Category - หมวดหมู่
model Category {
  id          Int    @id @default(autoincrement())
  name        String @unique
  description String?
  
  // Relations
  posts       Post[] // Category หนึ่งหมวดมีหลาย Posts
}

// โมเดล Post - โพสต์
model Post {
  id          Int      @id @default(autoincrement())
  title       String
  content     String?
  published   Boolean  @default(false)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  // Foreign Keys - กุญแจนอก
  authorId    Int      // เชื่อมกับ User
  categoryId  Int      // เชื่อมกับ Category
  
  // Relations
  author      User     @relation(fields: [authorId], references: [id])
  category    Category @relation(fields: [categoryId], references: [id])
}
```

## 📋 อธิบาย Field Types และ Attributes

### 🔤 Prisma Field Types

#### **Scalar Types** (ประเภทข้อมูลพื้นฐาน)

```prisma
model Example {
  id          Int       // จำนวนเต็ม
  email       String    // ข้อความ
  price       Float     // ทศนิยม
  isActive    Boolean   // true/false
  birthDate   DateTime  // วันที่และเวลา
  metadata    Json      // JSON object
}
```

#### **Field Modifiers** (ตัวปรับแต่ง)

```prisma
model Example {
  required    String    // บังคับต้องมีค่า
  optional    String?   // สามารถเป็น null ได้
  list        String[]  // Array ของ String
}
```

#### **Attributes** (คุณสมบัติพิเศษ)

```prisma
model Example {
  id          Int       @id                        // Primary key
  autoId      Int       @id @default(autoincrement()) // Auto increment
  uuid        String    @id @default(uuid())       // UUID
  email       String    @unique                    // ค่าไม่ซ้ำ
  createdAt   DateTime  @default(now())           // ค่าเริ่มต้น = เวลาปัจจุบัน
  updatedAt   DateTime  @updatedAt                // อัพเดทอัตโนมัติ
  customName  String    @map("custom_column_name") // ชื่อคอลัมน์ในฐานข้อมูล
}
```

#### **Index Attributes** (ดัชนี)

```prisma
model Example {
  userId      Int
  postId      Int
  
  @@unique([userId, postId])  // Composite unique
  @@index([userId])           // Index สำหรับค้นหาเร็วขึ้น
}
```

## 🔗 อธิบาย Relations (ความสัมพันธ์)

### 🔗 ความสัมพันธ์ใน Prisma

#### **One-to-Many** (หนึ่งต่อหลาย)

```prisma
model User {
  id    Int    @id @default(autoincrement())
  name  String
  posts Post[] // User หนึ่งคนมีหลาย Posts
}

model Post {
  id       Int    @id @default(autoincrement())
  title    String
  authorId Int    // Foreign key
  author   User   @relation(fields: [authorId], references: [id])
}
```

#### **One-to-One** (หนึ่งต่อหนึ่ง)

```prisma
model User {
  id      Int      @id @default(autoincrement())
  profile Profile? // User หนึ่งคนมี Profile หนึ่งอัน (หรือไม่มี)
}

model Profile {
  id     Int    @id @default(autoincrement())
  bio    String
  userId Int    @unique // ต้องเป็น unique
  user   User   @relation(fields: [userId], references: [id])
}
```

#### **Many-to-Many** (หลายต่อหลาย)

```prisma
model Post {
  id   Int   @id @default(autoincrement())
  tags Tag[] // Post หนึ่งโพสต์มีหลาย Tags
}

model Tag {
  id    Int    @id @default(autoincrement())
  name  String
  posts Post[] // Tag หนึ่งอันมีหลาย Posts
}

// Prisma จะสร้างตารางกลาง _PostToTag อัตโนมัติ
```

#### **การใช้ Explicit Many-to-Many**

```prisma
model Post {
  id       Int        @id @default(autoincrement())
  postTags PostTag[]  // ผ่านตารางกลาง
}

model Tag {
  id       Int        @id @default(autoincrement())
  postTags PostTag[]
}

model PostTag {
  postId Int
  tagId  Int
  post   Post @relation(fields: [postId], references: [id])
  tag    Tag  @relation(fields: [tagId], references: [id])
  
  @@id([postId, tagId]) // Composite primary key
}
```

## 🔄 รัน Migration สำหรับ Schema ใหม่

### 📋 ขั้นตอนการ Migration

```bash
# 1. สำคัญ! ลบข้อมูลเก่าก่อน (ถ้าต้องการเริ่มใหม่)
# rm prisma/dev.db
# rm -rf prisma/migrations

# 2. สร้าง migration ใหม่สำหรับ blog schema
npx prisma migrate dev --name blog-schema

# 3. Generate Prisma Client อีกครั้ง
npx prisma generate

# 4. เปิด Prisma Studio เพื่อดู schema
npx prisma studio
```

## 🌱 ทดสอบ Schema ด้วยการสร้างข้อมูล

### 📝 สร้างไฟล์ `src/seed.ts` เพื่อทดสอบ

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  console.log('🌱 Seeding database...');
  
  // สร้าง Categories
  const techCategory = await prisma.category.create({
    data: {
      name: 'Technology',
      description: 'Tech related posts',
    },
  });

  const lifestyleCategory = await prisma.category.create({
    data: {
      name: 'Lifestyle',
      description: 'Lifestyle and personal posts',
    },
  });

  // สร้าง User
  const user = await prisma.user.create({
    data: {
      email: 'john@example.com',
      name: 'John Doe',
    },
  });

  // สร้าง Posts
  const post1 = await prisma.post.create({
    data: {
      title: 'Getting Started with NestJS',
      content: 'NestJS is a powerful Node.js framework...',
      published: true,
      authorId: user.id,
      categoryId: techCategory.id,
    },
  });

  const post2 = await prisma.post.create({
    data: {
      title: 'Work-Life Balance Tips',
      content: 'Here are some tips for better work-life balance...',
      published: false,
      authorId: user.id,
      categoryId: lifestyleCategory.id,
    },
  });

  console.log('✅ Database seeded successfully!');
  console.log({ user, techCategory, lifestyleCategory, post1, post2 });
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

## 🧪 ทดสอบ Relations Queries

### 🔧 แก้ไข `app.controller.ts` - เพิ่ม routes ทดสอบ relations

```typescript
import { Controller, Get, Post, Param } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Controller()
export class AppController {
  constructor(private readonly prisma: PrismaService) {}

  // ดู User พร้อม Posts
  @Get('users/:id/posts')
  async getUserWithPosts(@Param('id') id: string) {
    return this.prisma.user.findUnique({
      where: { id: parseInt(id) },
      include: {
        posts: true, // รวม Posts ด้วย
      },
    });
  }

  // ดู Post พร้อม Author และ Category
  @Get('posts/:id/details')
  async getPostWithDetails(@Param('id') id: string) {
    return this.prisma.post.findUnique({
      where: { id: parseInt(id) },
      include: {
        author: true,   // รวม Author
        category: true, // รวม Category
      },
    });
  }

  // ดู Category พร้อม Posts ทั้งหมด
  @Get('categories/:id/posts')
  async getCategoryWithPosts(@Param('id') id: string) {
    return this.prisma.category.findUnique({
      where: { id: parseInt(id) },
      include: {
        posts: {
          include: {
            author: true, // รวม Author ของแต่ละ Post ด้วย
          },
        },
      },
    });
  }

  // สร้างข้อมูลทดสอบ
  @Post('seed')
  async seedData() {
    // ลบข้อมูลเก่า
    await this.prisma.post.deleteMany();
    await this.prisma.user.deleteMany();
    await this.prisma.category.deleteMany();

    // สร้างใหม่
    const category = await this.prisma.category.create({
      data: { name: 'Tech', description: 'Technology posts' },
    });

    const user = await this.prisma.user.create({
      data: { email: 'test@example.com', name: 'Test User' },
    });

    const post = await this.prisma.post.create({
      data: {
        title: 'Hello Prisma',
        content: 'This is my first post with Prisma!',
        published: true,
        authorId: user.id,
        categoryId: category.id,
      },
    });

    return { message: 'Data seeded!', user, category, post };
  }
}
```

## 🚀 รันและทดสอบ

### 📋 ขั้นตอนการทดสอบ

```bash
# รัน server
npm run start:dev
```

### 🧪 ทดสอบ API endpoints

| Method | Endpoint | หน้าที่ |
|--------|----------|---------|
| `POST` | `/seed` | สร้างข้อมูลทดสอบ |
| `GET` | `/users/1/posts` | ดู user และ posts ของเขา |
| `GET` | `/posts/1/details` | ดู post พร้อม author และ category |
| `GET` | `/categories/1/posts` | ดู category และ posts ในหมวดนั้น |

## 📚 สรุปบทที่ 3

### ✅ สิ่งที่เราได้เรียนรู้

- ✅ **เรียนรู้ Prisma Schema Language** - Field types, Attributes, Modifiers
- ✅ **สร้าง Relations** - One-to-Many, One-to-One, Many-to-Many
- ✅ **ใช้ Foreign Keys และ Reference fields**
- ✅ **ทดสอบ Relations Queries** ด้วย include
- ✅ **เข้าใจ Database Design** สำหรับระบบ Blog

### 🔑 Key Concepts

| แนวคิด | คำอธิบาย |
|--------|----------|
| **Field Types** | ประเภทข้อมูลพื้นฐานใน Prisma (Int, String, Boolean, DateTime, Json) |
| **Attributes** | คุณสมบัติพิเศษของ field (@id, @unique, @default, @updatedAt) |
| **Relations** | ความสัมพันธ์ระหว่างตาราง (One-to-Many, One-to-One, Many-to-Many) |
| **Foreign Keys** | คีย์ที่เชื่อมโยงระหว่างตาราง |
| **Include** | การดึงข้อมูลจาก relations พร้อมกับข้อมูลหลัก |

## 🚀 ในบทถัดไป

เราจะเรียนรู้ **NestJS Architecture อย่างลึกซึ้ง** - การสร้าง **Modules, Controllers, Services** และ **Dependency Injection** ในทางปฏิบัติ!

### 📋 สิ่งที่จะได้เรียนรู้

- 🏗️ **Module Pattern** และการจัดระเบียบโค้ด
- 🎮 **Controller Design** และ HTTP routing
- ⚙️ **Service Layer** และ business logic
- 🔌 **Dependency Injection** แบบ advanced
- 🧪 **Testing** modules และ services