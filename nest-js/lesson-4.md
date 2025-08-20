---
title: Lession 4
slug: example-post
date: 2020-11-05 11:20:38
draft: false
layout: ../layouts/PostLayout.astro
description: Eos eu docendi tractatos sapientem, brute option menandri in vix, quando vivendo accommodare te ius. Nec melius fastidii constituam id, viderer theophrastus ad sit, hinc semper periculis cum id. Noluisse postulant assentior est in
---

# 📖 บทที่ 4: NestJS Architecture - Module, Controller, Service

## 🎯 ทำความเข้าใจ NestJS Architecture

**NestJS** ใช้ Modular Architecture ที่แยกความรับผิดชอบออกเป็นชั้นๆ เหมือนสถาปัตยกรรมสมัยใหม่:

```
📦 Module (บ้าน)
├── 🎯 Controller (ประตูหน้าบ้าน - รับ HTTP requests)
├── ⚙️ Service (ห้องทำงาน - business logic)
└── 📊 Repository/Prisma (ห้องเก็บของ - database operations)
```

## 🚀 สร้าง Users Module แบบครบเครื่อง

### 📋 ใช้ NestJS CLI ช่วยสร้างไฟล์ให้อัตโนมัติ

```bash
# 1. สร้าง module, controller, service พร้อมกัน
nest generate resource users

# หรือสร้างทีละอัน:
# nest generate module users
# nest generate controller users  
# nest generate service users
```

### 📁 ไฟล์ที่จะถูกสร้าง

```
# src/users/
# ├── dto/
# │   ├── create-user.dto.ts
# │   └── update-user.dto.ts
# ├── entities/
# │   └── user.entity.ts
# ├── users.controller.spec.ts
# ├── users.controller.ts
# ├── users.module.ts
# ├── users.service.spec.ts
# └── users.service.ts
```

## 🛠️ สร้าง Users Module แบบแมนนวล

เพื่อให้เข้าใจลึกซึ้ง เรามาสร้างทีละไฟล์กันเองครับ:

### 🧩 `src/users/users.module.ts` - Module หลัก

```typescript
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { PrismaService } from '../prisma.service';

@Module({
  controllers: [UsersController], // ลงทะเบียน Controllers
  providers: [UsersService, PrismaService], // ลงทะเบียน Services
  exports: [UsersService], // ส่งออก Service ให้ modules อื่นใช้ได้
})
export class UsersModule {}
```

#### 📝 อธิบาย Module Configuration

| Property | ความหมาย |
|----------|----------|
| **controllers** | รายการ controllers ที่อยู่ใน module นี้ |
| **providers** | services, repositories ที่ใช้ใน module |
| **exports** | services ที่ module อื่นสามารถ import มาใช้ได้ |
| **imports** | modules อื่นที่เราต้องการใช้ |

## 📝 สร้าง DTOs (Data Transfer Objects)

### 🎯 DTOs ใช้สำหรับกำหนดรูปแบบข้อมูลที่รับ-ส่ง

#### 📤 `src/users/dto/create-user.dto.ts`

```typescript
export class CreateUserDto {
  email: string;
  name?: string; // optional
}
```

#### 🔄 `src/users/dto/update-user.dto.ts`

```typescript
export class UpdateUserDto {
  email?: string;
  name?: string;
  // ทุกฟิลด์เป็น optional เพราะ PATCH อาจอัพเดทบางฟิลด์
}
```

#### 📥 `src/users/dto/user-response.dto.ts`

```typescript
export class UserResponseDto {
  id: number;
  email: string;
  name: string | null;
  createdAt: Date;
  updatedAt: Date;
}
```

### 🤔 ทำไมต้องใช้ DTO?

- ✅ **Type Safety** - ป้องกัน runtime errors
- ✅ **Validation** - ตรวจสอบข้อมูลก่อนประมวลผล
- ✅ **Documentation** - เอกสาร API ชัดเจน
- ✅ **Transformation** - แปลงข้อมูลตามต้องการ

## ⚙️ สร้าง Users Service (Business Logic)

### 🔧 `src/users/users.service.ts`

```typescript
import { Injectable, NotFoundException, ConflictException, InternalServerErrorException } from '@nestjs/common';
import { PrismaService } from '../prisma.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Injectable()
export class UsersService {
  // Dependency Injection - ฉีด PrismaService เข้ามา
  constructor(private prisma: PrismaService) {}

  // สร้าง User ใหม่
  async create(createUserDto: CreateUserDto) {
    try {
      return await this.prisma.user.create({
        data: createUserDto,
      });
    } catch (error) {
      // จัดการ error เมื่อ email ซ้ำ
      if (error.code === 'P2002') {
        throw new ConflictException('Email already exists');
      }
      throw new InternalServerErrorException('Failed to create user');
    }
  }

  // ดู Users ทั้งหมด
  async findAll() {
    return this.prisma.user.findMany({
      orderBy: { createdAt: 'desc' }, // เรียงตามวันที่สร้างล่าสุด
    });
  }

  // ดู User คนเดียว
  async findOne(id: number) {
    const user = await this.prisma.user.findUnique({
      where: { id },
    });

    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }

    return user;
  }

  // แก้ไข User
  async update(id: number, updateUserDto: UpdateUserDto) {
    // ตรวจสอบว่า user มีอยู่จริงก่อน
    await this.findOne(id);

    try {
      return await this.prisma.user.update({
        where: { id },
        data: updateUserDto,
      });
    } catch (error) {
      if (error.code === 'P2002') {
        throw new ConflictException('Email already exists');
      }
      throw new InternalServerErrorException('Failed to update user');
    }
  }

  // ลบ User
  async remove(id: number) {
    // ตรวจสอบว่า user มีอยู่จริงก่อน
    await this.findOne(id);

    return this.prisma.user.delete({
      where: { id },
    });
  }

  // ดู User พร้อม Posts
  async findUserWithPosts(id: number) {
    const user = await this.prisma.user.findUnique({
      where: { id },
      include: {
        posts: {
          include: {
            category: true, // รวม category ของแต่ละ post ด้วย
          },
        },
      },
    });

    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }

    return user;
  }
}
```

## 🎮 สร้าง Users Controller (HTTP Layer)

### 🚪 `src/users/users.controller.ts`

```typescript
import {
  Controller,
  Get,
  Post,
  Body,
  Patch,
  Param,
  Delete,
  HttpStatus,
  HttpCode,
} from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Controller('users') // Base path: /users
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  // POST /users - สร้าง User ใหม่
  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  // GET /users - ดู Users ทั้งหมด
  @Get()
  findAll() {
    return this.usersService.findAll();
  }

  // GET /users/:id - ดู User คนเดียว
  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(+id); // +id แปลง string เป็น number
  }

  // GET /users/:id/posts - ดู User พร้อม Posts
  @Get(':id/posts')
  findUserWithPosts(@Param('id') id: string) {
    return this.usersService.findUserWithPosts(+id);
  }

  // PATCH /users/:id - แก้ไข User
  @Patch(':id')
  update(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
    return this.usersService.update(+id, updateUserDto);
  }

  // DELETE /users/:id - ลบ User
  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT) // ส่ง 204 แทน 200
  remove(@Param('id') id: string) {
    return this.usersService.remove(+id);
  }
}
```

### 🔍 อธิบาย Decorators

| Decorator | ความหมาย |
|-----------|----------|
| `@Controller('users')` | กำหนด base path |
| `@Get()`, `@Post()`, `@Patch()`, `@Delete()` | HTTP methods |
| `@Param('id')` | ดึงค่าจาก URL parameter |
| `@Body()` | ดึงข้อมูลจาก request body |
| `@HttpCode()` | กำหนด HTTP status code ที่ส่งกลับ |

## 🔗 ลงทะเบียน Module ใน App Module

### 🧩 `src/app.module.ts` - เพิ่ม UsersModule

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { PrismaService } from './prisma.service';
import { UsersModule } from './users/users.module'; // Import UsersModule

@Module({
  imports: [UsersModule], // เพิ่ม UsersModule ใน imports
  controllers: [AppController],
  providers: [AppService, PrismaService],
})
export class AppModule {}
```

> **💡 หมายเหตุ**: 
> - `imports` คือ modules อื่นที่เราต้องการใช้
> - `PrismaService` ใน providers หลักจะไม่ต้องการแล้ว
> - เพราะแต่ละ module จะมี PrismaService ของตัวเอง

---

## 🧪 ทดสอบ Users API

### 📋 ขั้นตอนการทดสอบ

```bash
# รันเซิร์ฟเวอร์
npm run start:dev
```

### 🧪 ทดสอบ API ด้วย curl หรือ Postman

| Method | Endpoint | หน้าที่ | ตัวอย่าง |
|--------|----------|---------|----------|
| `POST` | `/users` | สร้าง User ใหม่ | `{"email": "john@example.com", "name": "John Doe"}` |
| `GET` | `/users` | ดู Users ทั้งหมด | - |
| `GET` | `/users/:id` | ดู User คนเดียว | `/users/1` |
| `GET` | `/users/:id/posts` | ดู User พร้อม Posts | `/users/1/posts` |
| `PATCH` | `/users/:id` | แก้ไข User | `{"name": "John Smith"}` |
| `DELETE` | `/users/:id` | ลบ User | `/users/1` |

### 📝 ตัวอย่างการทดสอบด้วย curl

```bash
# 1. สร้าง User ใหม่ - POST /users
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"email": "john@example.com", "name": "John Doe"}'

# 2. ดู Users ทั้งหมด - GET /users
curl http://localhost:3000/users

# 3. ดู User คนเดียว - GET /users/:id
curl http://localhost:3000/users/1

# 4. ดู User พร้อม Posts - GET /users/:id/posts
curl http://localhost:3000/users/1/posts

# 5. แก้ไข User - PATCH /users/:id
curl -X PATCH http://localhost:3000/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name": "John Smith"}'

# 6. ลบ User - DELETE /users/:id
curl -X DELETE http://localhost:3000/users/1
```

## 🔌 เข้าใจ Dependency Injection

### 🔧 ตัวอย่าง Dependency Injection ใน NestJS

#### ❌ **Traditional way (ไม่ใช้ DI) - ไม่แนะนำ**

```typescript
class UserController {
  private userService: UserService;
  
  constructor() {
    this.userService = new UserService(); // สร้าง instance เอง
  }
}
```

#### ✅ **NestJS way (ใช้ DI) - แนะนำ**

```typescript
@Controller('users')
class UserController {
  // NestJS จะ inject UserService ให้อัตโนมัติ
  constructor(private readonly userService: UserService) {}
}
```

### 🎯 ข้อดีของ Dependency Injection

- ✅ **Easy Testing** - สามารถ mock services ได้ง่าย
- ✅ **Loose Coupling** - classes ไม่ผูกติดกันแน่น
- ✅ **Code Reuse** - สามารถใช้ service ใน controllers หลายตัว
- ✅ **Configuration** - NestJS จัดการ lifecycle ให้

### 🔄 ตัวอย่าง: ใช้ UserService ใน Controllers หลายตัว

```typescript
@Controller('users')
class UserController {
  constructor(private userService: UserService) {}
}

@Controller('admin')  
class AdminController {
  constructor(private userService: UserService) {} // ใช้ service เดียวกัน
}
```

> **💡 ทิป**: NestJS จะสร้าง UserService instance เดียว (Singleton) และ inject ให้ทุก controller ที่ต้องการ

## 📚 สรุปบทที่ 4

### ✅ สิ่งที่เราได้เรียนรู้

- ✅ **เข้าใจ NestJS Architecture** - Module, Controller, Service pattern
- ✅ **สร้าง Complete Users Module** ที่มีครบทุกส่วน
- ✅ **เรียนรู้ DTOs** สำหรับ type safety และ validation
- ✅ **ใช้ Dependency Injection** ในทางปฏิบัติ
- ✅ **สร้าง RESTful API endpoints** ที่สมบูรณ์
- ✅ **จัดการ Error Handling** และ HTTP status codes

### 🏗️ โครงสร้างที่ได้

```
src/
├── users/
│   ├── dto/
│   │   ├── create-user.dto.ts
│   │   └── update-user.dto.ts  
│   ├── users.controller.ts
│   ├── users.service.ts
│   └── users.module.ts
├── app.module.ts
└── prisma.service.ts
```

### 🔑 Key Concepts

| แนวคิด | คำอธิบาย |
|--------|----------|
| **Module Pattern** | การแบ่งแยกโค้ดเป็นส่วนๆ ที่ชัดเจนและสามารถนำกลับมาใช้ได้ |
| **Controller** | จัดการ HTTP requests และ responses |
| **Service** | จัดการ business logic และ database operations |
| **DTO** | Data Transfer Object สำหรับกำหนดรูปแบบข้อมูล |
| **Dependency Injection** | ระบบที่ NestJS จัดการ dependencies ให้อัตโนมัติ |


## 🚀 ในบทถัดไป

เราจะทำ **CRUD Operations แบบสมบูรณ์** พร้อมจัดการกับ **Relations, Error Handling, และ Advanced Queries**!

### 📋 สิ่งที่จะได้เรียนรู้

- 🔄 **Complete CRUD Operations** สำหรับทุก entities
- 🔗 **Relationship Management** ระหว่างตาราง
- 🛡️ **Advanced Error Handling** และ validation
- 🔍 **Complex Queries** และ filtering
- 📊 **Pagination** และ sorting