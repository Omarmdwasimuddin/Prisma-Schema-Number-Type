# 🔢 Prisma + PostgreSQL — Number Types

> **লক্ষ্য:** Prisma Schema-এ সংখ্যার বিভিন্ন ধরন কী, কখন কোনটা ব্যবহার করব — সেটা সহজভাবে বোঝা।

---


#### Prisma.Schema
```bash
generator client {
  provider = "prisma-client-js"
  output   = "../app/generated/prisma"
}

datasource db {
  provider = "postgresql"
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  
  // PostgreSQL compatible numeric types
  age       Int?
  points    BigInt   @default(0)
  rating    Decimal? @db.Decimal(10, 2)
  latitude  Float?   @db.DoublePrecision
  longitude Float?   @db.DoublePrecision

  // Metadata (Industry Standard)
  status    UserStatus @default(ACTIVE)
  createdAt DateTime   @default(now())
  updatedAt DateTime   @updatedAt

  @@map("users")
}

// Numeric Types Learning Lab
model NumericTypesLab {
  id Int @id @default(autoincrement())

  // 1. Integers (Purno Shonkha)
  smallInt  Int    @db.SmallInt    // Range: -32,768 to 32,767 (Choto shonkha, memory kom khay)
  normalInt Int                    // Range: -2.1B to 2.1B (Default Integer)
  bigInt    BigInt                 // Range: Very large numbers (ID ba Transaction amount er jonno)

  // 2. Floating Point (Doshomik - Precision ektu kom thakte pare)
  float     Float                  // Default Float (PostgreSQL e eta 'Double Precision' hoy)
  real      Float  @db.Real        // Kom precision er float (4 bytes)

  // 3. Fixed Precision (Taka-Poyshar jonno best)
  decimal   Decimal @db.Decimal(10, 2) // Total 10 digits, doshomik er pore 2 digit (Standard for Price)

  // 4. Special Attributes
  withDefault Int     @default(100)  // Default value set kora
  uniqueValue Int     @unique        // Duplicate value dewa jabe na
  optional    Int?                   // Nullable (Value na thakleo cholbe)

  @@map("numeric_types_lab")
}

enum UserStatus {
  ACTIVE
  INACTIVE
  PENDING
}

```
---


## 📌 দ্রুত রেফারেন্স টেবিল

| Prisma Type | PostgreSQL Type | বাংলা নাম | Range / বিবরণ | কখন ব্যবহার করবে |
|---|---|---|---|---|
| `Int` | `INTEGER` | পূর্ণসংখ্যা | -2.1B থেকে +2.1B | সাধারণ ID, বয়স, কাউন্ট |
| `Int @db.SmallInt` | `SMALLINT` | ছোট পূর্ণসংখ্যা | -32,768 থেকে 32,767 | কম range দরকার, memory বাঁচাতে |
| `BigInt` | `BIGINT` | বড় পূর্ণসংখ্যা | অনেক বড় সংখ্যা | Transaction ID, বড় কাউন্টার |
| `Float` | `DOUBLE PRECISION` | দশমিক (কম নির্ভুল) | 15-17 significant digits | সাধারণ দশমিক, latitude/longitude |
| `Float @db.Real` | `REAL` | দশমিক (আরো কম নির্ভুল) | 6 significant digits | কম precision হলেও চলে |
| `Decimal` | `DECIMAL(p, s)` | নির্ভুল দশমিক | নির্ধারিত digit সংখ্যা | টাকা-পয়সা, rating, finance |

---

## 1️⃣ Integer — পূর্ণসংখ্যা

```prisma
model NumericTypesLab {
  smallInt  Int    @db.SmallInt  // -32,768 থেকে 32,767
  normalInt Int                  // -2,147,483,648 থেকে 2,147,483,647
  bigInt    BigInt               // অনেক বড় সংখ্যা
}
```

### কোনটা কখন?

```
SmallInt  → স্টার রেটিং (1-5), মাস (1-12), ছোট কোড নম্বর
Int       → User ID, বয়স, পেজ নম্বর, product count
BigInt    → Transaction ID, timestamp milliseconds, বিশাল counter
```

> ⚠️ **সতর্কতা:** `BigInt` এ JavaScript-এ সরাসরি `number` হিসেবে কাজ করে না — `BigInt()` wrapper লাগে।

---

## 2️⃣ Float — দশমিক (আনুমানিক)

```prisma
model NumericTypesLab {
  float  Float            // Double Precision (PostgreSQL default)
  real   Float @db.Real   // Real / Single Precision (4 bytes, কম নির্ভুল)
}
```

### পার্থক্য কী?

| | `Float` (Double) | `Float @db.Real` |
|---|---|---|
| **Storage** | 8 bytes | 4 bytes |
| **নির্ভুলতা** | ~15 digits | ~6 digits |
| **ব্যবহার** | Latitude, Longitude | সাধারণ দশমিক যেখানে precision কম দরকার |

> ℹ️ Prisma-তে `Float` লিখলে PostgreSQL-এ সেটা `DOUBLE PRECISION` হয়ে যায়।

---

## 3️⃣ Decimal — নির্ভুল দশমিক (টাকার জন্য সেরা)

```prisma
model NumericTypesLab {
  decimal Decimal @db.Decimal(10, 2)
  //                          ↑   ↑
  //              মোট digit   দশমিকের পরে digit
}
```

### উদাহরণ: `@db.Decimal(10, 2)`

```
মোট 10টি digit, দশমিকের পরে 2টি
→ সর্বোচ্চ সংখ্যা: 99,999,999.99
→ উদাহরণ: 12345.67 ✅ | 12345678.90 ✅ | 1234567890.12 ❌ (10 digit পার)
```

### কেন Float নয়, Decimal ব্যবহার করব টাকার জন্য?

```js
// ❌ Float দিয়ে সমস্যা (Floating Point Error)
0.1 + 0.2 = 0.30000000000000004  // ভুল!

// ✅ Decimal দিয়ে সঠিক
Decimal("0.1") + Decimal("0.2") = 0.3  // সঠিক!
```

> 💡 **Rule of thumb:** টাকা, মূল্য, রেটিং, ভ্যাট — সব কিছুতে `Decimal` ব্যবহার করো।

---

## 4️⃣ Special Attributes — বিশেষ গুণাবলী

```prisma
model NumericTypesLab {
  withDefault Int     @default(100)  // ✅ value না দিলে 100 থাকবে
  uniqueValue Int     @unique        // ✅ duplicate value দেওয়া যাবে না
  optional    Int?                   // ✅ value না থাকলেও (NULL) চলবে
}
```

| Attribute | মানে কী | উদাহরণ |
|---|---|---|
| `@default(100)` | প্রথমে কোনো value না দিলে 100 বসবে | `points @default(0)` |
| `@unique` | একই value দুইবার রাখা যাবে না | `email @unique` |
| `?` (Optional) | NULL হতে পারবে | `age Int?` |
| `@id @default(autoincrement())` | নিজে নিজে 1, 2, 3... বাড়তে থাকবে | `id Int @id @default(autoincrement())` |

---

## 5️⃣ Real Project — User Model বিশ্লেষণ

```prisma
model User {
  id        Int      @id @default(autoincrement())  // Auto-increment Primary Key
  email     String   @unique                         // Duplicate email চলবে না

  age       Int?                                     // বয়স — optional (NULL হতে পারে)
  points    BigInt   @default(0)                     // পয়েন্ট — বড় হতে পারে, default 0
  rating    Decimal? @db.Decimal(10, 2)              // রেটিং — নির্ভুল দশমিক, optional
  latitude  Float?   @db.DoublePrecision             // অবস্থান — high precision দরকার
  longitude Float?   @db.DoublePrecision             // অবস্থান — high precision দরকার
}
```

### প্রতিটি সিদ্ধান্তের কারণ:

```
age      → Int?       কারণ: বয়স পূর্ণসংখ্যা, সবসময় থাকে না
points   → BigInt     কারণ: পয়েন্ট অনেক বড় হতে পারে, default 0
rating   → Decimal    কারণ: 4.75 এর মতো নির্ভুল মান দরকার
lat/long → Float      কারণ: 23.8103° এর মতো দশমিক, খুব বড় precision দরকার
```

---

## 6️⃣ Enum — নির্দিষ্ট মান (Status)

```prisma
enum UserStatus {
  ACTIVE    // সক্রিয় ব্যবহারকারী
  INACTIVE  // নিষ্ক্রিয়
  PENDING   // অপেক্ষমাণ (যেমন: email verify করেনি)
}

model User {
  status UserStatus @default(ACTIVE)  // নতুন user সবসময় ACTIVE হবে
}
```

> ✅ **Enum কেন ভালো?** Database-এ শুধু নির্দিষ্ট মানগুলোই রাখা যাবে — typo বা ভুল value ঢুকতে পারবে না।

---

## 🧠 মনে রাখার সহজ নিয়ম

```
পূর্ণসংখ্যা (ভাঙা না) → Int / BigInt / SmallInt
টাকা-পয়সা / রেটিং   → Decimal  (নির্ভুল!)
অবস্থান / বৈজ্ঞানিক  → Float    (আনুমানিক, তবে range বড়)
নির্দিষ্ট কিছু মান   → Enum
```

---

## 📋 সংক্ষিপ্ত চিট শিট

```prisma
// 🔢 সব ধরনের সংখ্যা একনজরে
id        Int     @id @default(autoincrement())  // Primary Key
age       Int?                                   // Optional Integer
smallNum  Int     @db.SmallInt                   // ছোট Integer (memory সাশ্রয়)
bigNum    BigInt  @default(0)                    // বড় Integer
price     Decimal @db.Decimal(10, 2)             // টাকা (নির্ভুল)
lat       Float   @db.DoublePrecision            // দশমিক (উচ্চ নির্ভুলতা)
score     Float   @db.Real                       // দশমিক (কম নির্ভুলতা)
```

---
