---
title: Roadmap
slug: example-post
date: 2020-11-05 11:20:38
draft: false
layout: ../layouts/PostLayout.astro
description: Eos eu docendi tractatos sapientem, brute option menandri in vix, quando vivendo accommodare te ius. Nec melius fastidii constituam id, viderer theophrastus ad sit, hinc semper periculis cum id. Noluisse postulant assentior est in
---

# 🚀 NestJS + Prisma Learning Roadmap

##### เป้าหมาย: เรียนรู้การสร้าง RESTful API ด้วย NestJS และ Prisma ตั้งแต่พื้นฐานจนถึงระดับกลาง

---

## 📖 บทที่ 1: รู้จักและติดตั้ง NestJS

### 🎯 สิ่งที่จะได้เรียนรู้
- **NestJS คืออะไร** และทำไมถึงนิยมในหมู่ developers
- **สร้างโปรเจกต์ NestJS แรก** พร้อมการตั้งค่าพื้นฐาน
- **ทำความเข้าใจโครงสร้างโปรเจกต์** และไฟล์สำคัญต่างๆ
- **รัน Hello World แรก** เพื่อทดสอบการทำงาน

### ⏱️ เวลาที่ใช้: 2-3 ชั่วโมง

---

## 🗄️ บทที่ 2: เริ่มต้นกับ Prisma

### 🎯 สิ่งที่จะได้เรียนรู้
- **Prisma คืออะไร** และทำงานอย่างไร
- **ติดตั้งและ config Prisma** ในโปรเจกต์ NestJS
- **เชื่อมต่อกับฐานข้อมูล** (เริ่มต้นด้วย SQLite เพื่อความง่าย)
- **สร้าง Schema แรก** และเข้าใจโครงสร้าง

### ⏱️ เวลาที่ใช้: 2-3 ชั่วโมง

---

## 🏗️ บทที่ 3: ออกแบบ Database Schema

### 🎯 สิ่งที่จะได้เรียนรู้
- **เรียนรู้ Prisma Schema Language** และ syntax
- **กำหนด Model, Field Types, Relations** ระหว่างตาราง
- **Migration และ Generate Prisma Client** เพื่อใช้งาน
- **ทดสอบการเชื่อมต่อ** และการทำงานของ schema

### ⏱️ เวลาที่ใช้: 3-4 ชั่วโมง

---

## 🏛️ บทที่ 4: NestJS Architecture - Module, Controller, Service

### 🎯 สิ่งที่จะได้เรียนรู้
- **Module Pattern ใน NestJS** และการจัดระเบียบโค้ด
- **สร้าง Controller และ HTTP Routes** สำหรับ API endpoints
- **สร้าง Service และ Business Logic** แยกจาก controller
- **Dependency Injection ในทางปฏิบัติ** และการใช้งาน

### ⏱️ เวลาที่ใช้: 3-4 ชั่วโมง

---

## 🔄 บทที่ 5: ทำ CRUD Operations

### 🎯 สิ่งที่จะได้เรียนรู้
- **CREATE** - เพิ่มข้อมูลใหม่ลงฐานข้อมูล
- **READ** - อ่านข้อมูล (หนึ่งรายการและหลายรายการ)
- **UPDATE** - แก้ไขข้อมูลที่มีอยู่
- **DELETE** - ลบข้อมูลออกจากฐานข้อมูล
- **จัดการ Error Handling** และ HTTP status codes

### ⏱️ เวลาที่ใช้: 4-5 ชั่วโมง

---

## ⚡ บทที่ 6: Advanced Features และ Best Practices

### 🎯 สิ่งที่จะได้เรียนรู้
- **Validation ด้วย DTOs** เพื่อตรวจสอบข้อมูล input
- **Custom Decorators** สำหรับเพิ่มฟังก์ชันการทำงาน
- **Exception Filters** เพื่อจัดการ error แบบ centralized
- **การทำ Relationship Queries** ระหว่างตารางต่างๆ
- **Tips และ Best Practices** สำหรับการพัฒนา production