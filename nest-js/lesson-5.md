# 📖 บทที่ 5: ทำ CRUD Operations แบบสมบูรณ์

## 🚀 สร้าง Posts Module สำหรับ CRUD ครบเครื่อง

มาสร้าง Posts Module ที่มี CRUD operations แบบละเอียดกันครับ รวมถึงการจัดการ Relations:

### 📝 DTOs สำหรับ Posts

#### 📤 `src/posts/dto/create-post.dto.ts`

```typescript
export class CreatePostDto {
  title: string;
  content?: string;
  published?: boolean;
  authorId: number;    // Foreign key ถึง User
  categoryId: number;  // Foreign key ถึง Category
}
```

#### 🔄 `src/posts/dto/update-post.dto.ts`

```typescript
export class UpdatePostDto {
  title?: string;
  content?: string;
  published?: boolean;
  categoryId?: number; // สามารถเปลี่ยนหมวดหมู่ได้
}
```

#### 🔍 `src/posts/dto/query-post.dto.ts` - สำหรับ filtering

```typescript
export class QueryPostDto {
  published?: boolean;  // กรองตาม published
  authorId?: number;    // กรองตาม author
  categoryId?: number;  // กรองตาม category
  search?: string;      // ค้นหาใน title หรือ content
  page?: number;        // pagination
  limit?: number;       // จำนวนต่อหน้า
}
```

#### 📥 `src/posts/dto/post-response.dto.ts`

```typescript
export class PostResponseDto {
  id: number;
  title: string;
  content: string | null;
  published: boolean;
  createdAt: Date;
  updatedAt: Date;
  author: {
    id: number;
    name: string | null;
    email: string;
  };
  category: {
    id: number;
    name: string;
  };
}
```

## ⚙️ Posts Service - Business Logic ที่สมบูรณ์

### 🔧 `src/posts/posts.service.ts`

```typescript
import { 
  Injectable, 
  NotFoundException, 
  BadRequestException,
  ConflictException,
  InternalServerErrorException
} from '@nestjs/common';
import { PrismaService } from '../prisma.service';
import { CreatePostDto } from './dto/create-post.dto';
import { UpdatePostDto } from './dto/update-post.dto';
import { QueryPostDto } from './dto/query-post.dto';

@Injectable()
export class PostsService {
  constructor(private prisma: PrismaService) {}

  // CREATE - สร้าง Post ใหม่
  async create(createPostDto: CreatePostDto) {
    // ตรวจสอบว่า Author และ Category มีอยู่จริง
    const [author, category] = await Promise.all([
      this.prisma.user.findUnique({ where: { id: createPostDto.authorId } }),
      this.prisma.category.findUnique({ where: { id: createPostDto.categoryId } })
    ]);

    if (!author) {
      throw new BadRequestException('Author not found');
    }
    if (!category) {
      throw new BadRequestException('Category not found');
    }

    try {
      return await this.prisma.post.create({
        data: createPostDto,
        include: {
          author: { select: { id: true, name: true, email: true } },
          category: { select: { id: true, name: true } }
        }
      });
    } catch (error) {
      if (error.code === 'P2002') {
        throw new ConflictException('Post with this title already exists');
      }
      if (error.code === 'P2003') {
        throw new BadRequestException('Invalid author or category reference');
      }
      throw new InternalServerErrorException('Failed to create post');
    }
  }

  // READ - ดู Posts ทั้งหมดพร้อม filtering และ pagination
  async findAll(queryDto: QueryPostDto = {}) {
    const {
      published,
      authorId,
      categoryId,
      search,
      page = 1,
      limit = 10
    } = queryDto;

    // สร้าง where conditions
    const where: any = {};

    if (published !== undefined) {
      where.published = published;
    }
    if (authorId) {
      where.authorId = authorId;
    }
    if (categoryId) {
      where.categoryId = categoryId;
    }
    if (search) {
      where.OR = [
        { title: { contains: search, mode: 'insensitive' } },
        { content: { contains: search, mode: 'insensitive' } }
      ];
    }

    // Pagination
    const skip = (page - 1) * limit;

    const [posts, total] = await Promise.all([
      this.prisma.post.findMany({
        where,
        skip,
        take: limit,
        orderBy: { createdAt: 'desc' },
        include: {
          author: { select: { id: true, name: true, email: true } },
          category: { select: { id: true, name: true } }
        }
      }),
      this.prisma.post.count({ where })
    ]);

    return {
      posts,
      pagination: {
        total,
        page,
        limit,
        totalPages: Math.ceil(total / limit)
      }
    };
  }

  // READ - ดู Post เดียวพร้อม Relations
  async findOne(id: number) {
    const post = await this.prisma.post.findUnique({
      where: { id },
      include: {
        author: { select: { id: true, name: true, email: true } },
        category: { select: { id: true, name: true, description: true } }
      }
    });

    if (!post) {
      throw new NotFoundException(`Post with ID ${id} not found`);
    }

    return post;
  }

  // UPDATE - แก้ไข Post
  async update(id: number, updatePostDto: UpdatePostDto) {
    // ตรวจสอบว่า post มีอยู่จริง
    await this.findOne(id);

    // ถ้ามีการเปลี่ยน categoryId ต้องตรวจสอบว่า category มีอยู่
    if (updatePostDto.categoryId) {
      const category = await this.prisma.category.findUnique({
        where: { id: updatePostDto.categoryId }
      });
      if (!category) {
        throw new BadRequestException('Category not found');
      }
    }

    try {
      return await this.prisma.post.update({
        where: { id },
        data: updatePostDto,
        include: {
          author: { select: { id: true, name: true, email: true } },
          category: { select: { id: true, name: true } }
        }
      });
    } catch (error) {
      if (error.code === 'P2002') {
        throw new ConflictException('Post with this title already exists');
      }
      if (error.code === 'P2003') {
        throw new BadRequestException('Invalid category reference');
      }
      throw new InternalServerErrorException('Failed to update post');
    }
  }

  // DELETE - ลบ Post
  async remove(id: number) {
    // ตรวจสอบว่า post มีอยู่จริง
    await this.findOne(id);

    try {
      return await this.prisma.post.delete({
        where: { id }
      });
    } catch (error) {
      if (error.code === 'P2003') {
        throw new BadRequestException('Cannot delete post with existing references');
      }
      throw new InternalServerErrorException('Failed to delete post');
    }
  }

  // Advanced Operations

  // เปลี่ยนสถานะ publish/unpublish
  async togglePublish(id: number) {
    const post = await this.findOne(id);
    
    return this.prisma.post.update({
      where: { id },
      data: { published: !post.published },
      include: {
        author: { select: { id: true, name: true, email: true } },
        category: { select: { id: true, name: true } }
      }
    });
  }

  // ดู Posts ยอดนิยม (มากที่สุด 5 อัน)
  async getPopularPosts() {
    return this.prisma.post.findMany({
      where: { published: true },
      take: 5,
      orderBy: { createdAt: 'desc' },
      include: {
        author: { select: { id: true, name: true } },
        category: { select: { name: true } }
      }
    });
  }

  // สถิติ Posts
  async getPostStats() {
    const [total, published, unpublished, byCategory] = await Promise.all([
      this.prisma.post.count(),
      this.prisma.post.count({ where: { published: true } }),
      this.prisma.post.count({ where: { published: false } }),
      this.prisma.post.groupBy({
        by: ['categoryId'],
        _count: { categoryId: true },
        include: {
          category: { select: { name: true } }
        }
      })
    ]);

    return {
      total,
      published,
      unpublished,
      byCategory
    };
  }
}
```

## 🎮 Posts Controller - HTTP Layer ที่ครบเครื่อง

### 🚪 `src/posts/posts.controller.ts`

```typescript
import {
  Controller,
  Get,
  Post,
  Body,
  Patch,
  Param,
  Delete,
  Query,
  HttpStatus,
  HttpCode,
  ParseIntPipe
} from '@nestjs/common';
import { PostsService } from './posts.service';
import { CreatePostDto } from './dto/create-post.dto';
import { UpdatePostDto } from './dto/update-post.dto';
import { QueryPostDto } from './dto/query-post.dto';

@Controller('posts')
export class PostsController {
  constructor(private readonly postsService: PostsService) {}

  // POST /posts - สร้าง Post ใหม่
  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() createPostDto: CreatePostDto) {
    return this.postsService.create(createPostDto);
  }

  // GET /posts - ดู Posts ทั้งหมดพร้อม filtering
  @Get()
  findAll(@Query() queryDto: QueryPostDto) {
    // แปลง string queries เป็น numbers/booleans
    if (queryDto.page) queryDto.page = +queryDto.page;
    if (queryDto.limit) queryDto.limit = +queryDto.limit;
    if (queryDto.authorId) queryDto.authorId = +queryDto.authorId;
    if (queryDto.categoryId) queryDto.categoryId = +queryDto.categoryId;
    if (queryDto.published !== undefined) {
      queryDto.published = queryDto.published === 'true';
    }

    return this.postsService.findAll(queryDto);
  }

  // GET /posts/popular - ดู Posts ยอดนิยม
  @Get('popular')
  getPopular() {
    return this.postsService.getPopularPosts();
  }

  // GET /posts/stats - สถิติ Posts
  @Get('stats')
  getStats() {
    return this.postsService.getPostStats();
  }

  // GET /posts/:id - ดู Post เดียว
  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.postsService.findOne(id);
  }

  // PATCH /posts/:id - แก้ไข Post
  @Patch(':id')
  update(
    @Param('id', ParseIntPipe) id: number,
    @Body() updatePostDto: UpdatePostDto
  ) {
    return this.postsService.update(id, updatePostDto);
  }

  // PATCH /posts/:id/toggle-publish - เปลี่ยนสถานะ publish
  @Patch(':id/toggle-publish')
  togglePublish(@Param('id', ParseIntPipe) id: number) {
    return this.postsService.togglePublish(id);
  }

  // DELETE /posts/:id - ลบ Post
  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id', ParseIntPipe) id: number) {
    return this.postsService.remove(id);
  }
}
```

### 🔍 อธิบาย Decorators เพิ่มเติม

| Decorator | ความหมาย |
|-----------|----------|
| `@Query()` | ดึงค่าจาก query parameters (?page=1&limit=10) |
| `ParseIntPipe` | แปลง string เป็น number และ validate อัตโนมัติ |
| `@Param('id', ParseIntPipe)` | ดึง id จาก URL และแปลงเป็น number |


## 🏷️ สร้าง Categories Module สำหรับ Reference Data

### 📝 DTOs และ Service

#### 📤 `src/categories/dto/create-category.dto.ts`

```typescript
export class CreateCategoryDto {
  name: string;
  description?: string;
}
```

#### ⚙️ `src/categories/categories.service.ts`

```typescript
import { Injectable, NotFoundException, ConflictException, InternalServerErrorException } from '@nestjs/common';
import { PrismaService } from '../prisma.service';
import { CreateCategoryDto } from './dto/create-category.dto';

@Injectable()
export class CategoriesService {
  constructor(private prisma: PrismaService) {}

  async create(createCategoryDto: CreateCategoryDto) {
    try {
      return await this.prisma.category.create({
        data: createCategoryDto
      });
    } catch (error) {
      if (error.code === 'P2002') {
        throw new ConflictException('Category name already exists');
      }
      throw new InternalServerErrorException('Failed to create category');
    }
  }

  async findAll() {
    return this.prisma.category.findMany({
      include: {
        _count: {
          select: { posts: true } // นับจำนวน posts ในแต่ละ category
        }
      },
      orderBy: { name: 'asc' }
    });
  }

  async findOne(id: number) {
    const category = await this.prisma.category.findUnique({
      where: { id },
      include: {
        posts: {
          where: { published: true }, // แสดงเฉพาะ posts ที่ publish แล้ว
          include: {
            author: { select: { name: true } }
          }
        }
      }
    });

    if (!category) {
      throw new NotFoundException(`Category with ID ${id} not found`);
    }

    return category;
  }
}
```

#### 🎮 `src/categories/categories.controller.ts`

```typescript
import { Controller, Get, Post, Body, Param, ParseIntPipe } from '@nestjs/common';
import { CategoriesService } from './categories.service';
import { CreateCategoryDto } from './dto/create-category.dto';

@Controller('categories')
export class CategoriesController {
  constructor(private readonly categoriesService: CategoriesService) {}

  @Post()
  create(@Body() createCategoryDto: CreateCategoryDto) {
    return this.categoriesService.create(createCategoryDto);
  }

  @Get()
  findAll() {
    return this.categoriesService.findAll();
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.categoriesService.findOne(id);
  }
}
```

## 🛡️ Error Handling และ Custom Exceptions

### 🔧 Custom Exception Filter

#### 📝 `src/common/exceptions/prisma-exception.filter.ts`

```typescript
import { ArgumentsHost, Catch, ExceptionFilter, HttpStatus } from '@nestjs/common';
import { Prisma } from '@prisma/client';
import { Response } from 'express';

@Catch(Prisma.PrismaClientKnownRequestError)
export class PrismaExceptionFilter implements ExceptionFilter {
  catch(exception: Prisma.PrismaClientKnownRequestError, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    
    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = 'Internal server error';

    switch (exception.code) {
      case 'P2002': // Unique constraint violation
        status = HttpStatus.CONFLICT;
        message = 'Resource already exists';
        break;
      case 'P2025': // Record not found
        status = HttpStatus.NOT_FOUND;
        message = 'Resource not found';
        break;
      case 'P2003': // Foreign key constraint
        status = HttpStatus.BAD_REQUEST;
        message = 'Invalid reference';
        break;
    }

    response.status(status).json({
      statusCode: status,
      message,
      error: exception.code,
      timestamp: new Date().toISOString(),
    });
  }
}
```

#### 📊 `src/common/dto/api-response.dto.ts`

```typescript
export class ApiResponseDto<T> {
  success: boolean;
  data?: T;
  message?: string;
  error?: string;
}
```

#### 🚀 ใน `main.ts` - ใช้ global exception filter

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { PrismaExceptionFilter } from './common/exceptions/prisma-exception.filter';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // ใช้ global exception filter
  app.useGlobalFilters(new PrismaExceptionFilter());
  
  await app.listen(3000);
}
bootstrap();
```

## 🧪 ทดสอบ CRUD Operations ครบเครื่อง

### 📋 เตรียมข้อมูลทดสอบ

```bash
# 1. สร้าง Categories
curl -X POST http://localhost:3000/categories \
  -H "Content-Type: application/json" \
  -d '{"name": "Technology", "description": "Tech posts"}'

curl -X POST http://localhost:3000/categories \
  -H "Content-Type: application/json" \
  -d '{"name": "Lifestyle", "description": "Life posts"}'

# 2. สร้าง Users  
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"email": "author@example.com", "name": "Author Name"}'

# 3. สร้าง Posts
curl -X POST http://localhost:3000/posts \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Getting Started with NestJS",
    "content": "NestJS is amazing framework...",
    "published": true,
    "authorId": 1,
    "categoryId": 1
  }'

curl -X POST http://localhost:3000/posts \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Work Life Balance",
    "content": "Tips for better work life balance...",
    "published": false,
    "authorId": 1,
    "categoryId": 2
  }'
```

### 🔍 ทดสอบ READ Operations

```bash
# ดู Posts ทั้งหมด
curl "http://localhost:3000/posts"

# ดู Posts ที่ publish แล้วเท่านั้น
curl "http://localhost:3000/posts?published=true"

# ค้นหา Posts
curl "http://localhost:3000/posts?search=nestjs"

# Pagination
curl "http://localhost:3000/posts?page=1&limit=5"

# กรองตาม author
curl "http://localhost:3000/posts?authorId=1"

# ดู Posts ยอดนิยม
curl "http://localhost:3000/posts/popular"

# ดูสถิติ
curl "http://localhost:3000/posts/stats"
```

### 🔄 ทดสอบ UPDATE Operations

```bash
# แก้ไข Post
curl -X PATCH http://localhost:3000/posts/1 \
  -H "Content-Type: application/json" \
  -d '{"title": "Updated: Getting Started with NestJS"}'

# เปลี่ยนสถานะ publish
curl -X PATCH http://localhost:3000/posts/2/toggle-publish
```

### 🗑️ ทดสอบ DELETE Operations

```bash
curl -X DELETE http://localhost:3000/posts/2
```

### 🔗 ทดสอบ Relations

```bash
# ดู User พร้อม Posts
curl "http://localhost:3000/users/1/posts"

# ดู Category พร้อม Posts
curl "http://localhost:3000/categories/1"
```

## 🔗 อัพเดท App Module ให้ครบ

### 🧩 `src/app.module.ts` - Module หลักที่สมบูรณ์

```typescript
import { Module } from '@nestjs/common';
import { UsersModule } from './users/users.module';
import { PostsModule } from './posts/posts.module';
import { CategoriesModule } from './categories/categories.module';

@Module({
  imports: [
    UsersModule,
    PostsModule,
    CategoriesModule,
  ],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

### 📦 `src/posts/posts.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { PostsService } from './posts.service';
import { PostsController } from './posts.controller';
import { PrismaService } from '../prisma.service';

@Module({
  controllers: [PostsController],
  providers: [PostsService, PrismaService],
  exports: [PostsService],
})
export class PostsModule {}
```

### 🏷️ `src/categories/categories.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { CategoriesService } from './categories.service';
import { CategoriesController } from './categories.controller';
import { PrismaService } from '../prisma.service';

@Module({
  controllers: [CategoriesController],
  providers: [CategoriesService, PrismaService],
  exports: [CategoriesService],
})
export class CategoriesModule {}
```

## 📚 สรุปบทที่ 5

### ✅ สิ่งที่เราได้เรียนรู้

- ✅ **CRUD Operations สมบูรณ์** - Create, Read, Update, Delete ทุก operations
- ✅ **Advanced Querying** - Filtering, Searching, Pagination, Sorting
- ✅ **Relations Handling** - การจัดการ Foreign Keys และ Joins
- ✅ **Error Handling** - Custom exceptions และ Prisma error handling
- ✅ **Business Logic** - Toggle publish, Statistics, Popular posts
- ✅ **Type Safety** - DTOs และ validation ครบถ้วน

### 🚀 API Endpoints ที่ได้

#### 👥 **Users**
```
├── POST   /users
├── GET    /users
├── GET    /users/:id
├── GET    /users/:id/posts
├── PATCH  /users/:id
└── DELETE /users/:id
```

#### 📝 **Posts**
```
├── POST   /posts
├── GET    /posts (with filtering)
├── GET    /posts/popular
├── GET    /posts/stats
├── GET    /posts/:id
├── PATCH  /posts/:id
├── PATCH  /posts/:id/toggle-publish
└── DELETE /posts/:id
```

#### 🏷️ **Categories**
```
├── POST   /categories
├── GET    /categories
└── GET    /categories/:id
```

### 🔑 Key Concepts

| แนวคิด | คำอธิบาย |
|--------|----------|
| **Advanced Querying** | การกรอง ค้นหา และแบ่งหน้าข้อมูล |
| **Relations** | การจัดการความสัมพันธ์ระหว่างตาราง |
| **Error Handling** | การจัดการข้อผิดพลาดแบบ centralized |
| **Business Logic** | ตรรกะทางธุรกิจที่ซับซ้อน |
| **Type Safety** | การป้องกันข้อผิดพลาดด้วย TypeScript |

## 🚀 ในบทสุดท้าย

เราจะเรียนรู้ **Advanced Features** - Validation, Custom Decorators, Exception Filters, และ **Best Practices** สำหรับการ deploy!

### 📋 สิ่งที่จะได้เรียนรู้

- 🛡️ **Validation** ด้วย class-validator และ pipes
- ✨ **Custom Decorators** สำหรับเพิ่มฟังก์ชันการทำงาน
- 🔧 **Exception Filters** แบบ advanced
- 🚀 **Best Practices** สำหรับ production
- 📦 **Deployment** และ CI/CD