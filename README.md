## Prisma-Schema-Number-Type

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
