---
theme: default
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Drizzle ORM Presentation
  A modern TypeScript ORM
drawings:
  persist: false
transition: slide-left
title: "Drizzle ORM: A Modern TypeScript ORM"
---

<Logo />

# Drizzle ORM

A Modern TypeScript ORM

---
layout: center
---

# Installation

### Install Drizzle ORM with PostgreSQL:

```shell {1|2}
npm i drizzle-orm pg
npm i -D @types/pg drizzle-kit
```

---

# Configuration

### Basic setup for Drizzle ORM with PostgreSQL:

```ts {1|2|3-10|12|13}
import { drizzle } from "drizzle-orm/node-postgres";
import { Client } from "pg";

const client = new Client({
  host: "127.0.0.1",
  port: 5432,
  user: "postgres",
  password: "password",
  database: "db_name",
});

await client.connect();
const db = drizzle(client);
```

---

# Schema

```ts
import { serial, text, pgTable, pgSchema } from "drizzle-orm/pg-core";

export const mySchema = pgSchema("my_schema");

export const colors = mySchema.enum('colors', ['red', 'green', 'blue']);

export const mySchemaUsers = mySchema.table('users', {
  id: serial('id').primaryKey(),
  name: text('name'),
  color: colors('color').default('red'),
});
```

### Equivalent SQL

```sql
CREATE SCHEMA "my_schema";
CREATE TYPE "my_schema"."colors" AS ENUM ('red', 'green', 'blue');
CREATE TABLE "my_schema"."users" (
  "id" serial PRIMARY KEY,
  "name" text,
  "color" "my_schema"."colors" DEFAULT 'red'
);
```

---

# Basic Queries - Part 1

### Define a table:

```ts {1|all}
import { pgTable, serial, text, integer } from "drizzle-orm/pg-core";

export const users = pgTable("users", {
  id: serial("id").primaryKey(),
  fullName: text("full_name"),
  age: integer("age"),
});
```

### Find One:

```ts
const user = await db.select().from(users).where(eq(users.id, 1)).get();
```

### Find All:

```ts
const allUsers = await db.select().from(users).all();
```

---

# Basic Queries - Part 2

### Insert:

```ts {2|3-6|all}
const newUser = await db
  .insert(users)
  .values({
    fullName: "John Doe",
    age: 30,
  })
  .returning();
```

### Update:

```ts {2|3|4|all}
const updatedUser = await db
  .update(users)
  .set({ age: 31 })
  .where(eq(users.id, 1))
  .returning();
```

### Delete:

```ts
const deletedUser = await db.delete(users).where(eq(users.id, 1)).returning();
```

---

# Basic Relationships (One-to-Many)

### Define related tables:

```ts {9-12|17|all}
import {
  pgTable,
  serial,
  text,
  integer,
  foreignKey,
} from "drizzle-orm/pg-core";

export const users = pgTable("users", {
  id: serial("id").primaryKey(),
  fullName: text("full_name"),
});

export const posts = pgTable("posts", {
  id: serial("id").primaryKey(),
  title: text("title"),
  authorId: integer("author_id").references(() => users.id),
});
```

---

# Query related data:

```ts {8|all}
const usersWithPosts = await db
  .select({
    id: users.id,
    name: users.fullName,
    posts: posts,
  })
  .from(users)
  .leftJoin(posts, eq(users.id, posts.authorId))
  .all();
```

---

# Many-to-Many Relationships

### Define tables for a many-to-many relationship:

```ts {22|23|all}{maxHeight:'400px'}
import {
  pgTable,
  serial,
  text,
  integer,
  primaryKey,
} from "drizzle-orm/pg-core";

export const users = pgTable("users", {
  id: serial("id").primaryKey(),
  name: text("name"),
});

export const categories = pgTable("categories", {
  id: serial("id").primaryKey(),
  name: text("name"),
});

export const usersToCategories = pgTable(
  "users_to_categories",
  {
    userId: integer("user_id").references(() => users.id),
    categoryId: integer("category_id").references(() => categories.id),
  },
  (t) => ({
    pk: primaryKey(t.userId, t.categoryId),
  }),
);
```

---

# Query many-to-many relationship:

```ts {8|9|all}
const usersWithCategories = await db
  .select({
    userId: users.id,
    userName: users.name,
    categories: sql<string>`array_agg(${categories.name})`,
  })
  .from(users)
  .leftJoin(usersToCategories, eq(users.id, usersToCategories.userId))
  .leftJoin(categories, eq(usersToCategories.categoryId, categories.id))
  .groupBy(users.id, users.name)
  .all();
```

---

# Unique Features

<v-clicks>

1. **Type Safety**: Full type safety and autocompletion
2. **SQL-like Query Builder**: Intuitive for SQL users
3. **Custom SQL**: Write raw SQL when needed
    ### Example of Custom SQL:
    
    ```ts
    const result = await db.execute(
      sql`SELECT * FROM ${users} WHERE ${users.id} = ${userId}`,
    );
    ```
4. **Relational Queries**: Complex relational queries support
5. **Extensions**: drizzle-zod, drizzle-valibot, prisma, ESLint

</v-clicks>

<v-click>

</v-click>

---

# Migrations

### Generate Migrations:

```shell
npx drizzle-kit generate:pg
```

### Will generate a migration file in drizzle folder

```sql
CREATE TABLE `users` (
 `id` bigint primary key auto_increment,
 `full_name` varchar(256)
);


CREATE TABLE `auth_otp` (
 `id` bigint primary key auto_increment,
 `phone` varchar(256),
 `user_id` int
);


ALTER TABLE auth_otp ADD CONSTRAINT auth_otp_user_id_users_id_fk FOREIGN KEY (`user_id`) REFERENCES users(`id`) ;
CREATE INDEX name_idx ON users (`full_name`);
```

---

# Apply Migrations:

```ts {2|10|all}
import { drizzle } from 'drizzle-orm/node-postgres';
import { migrate } from 'drizzle-orm/node-postgres/migrator';
import { Client } from 'pg';

const client = new Client({...});
await client.connect();

const db = drizzle(client);

await migrate(db, { migrationsFolder: './drizzle' });
```

---

# Conclusion

<v-clicks>

- Type-safe and intuitive ORM for TypeScript
- SQL-like syntax for easy adoption
- Powerful relationship handling
- Built-in migration tools
- Strong contender in the ORM landscape

</v-clicks>

---

# Thank You!

Any questions?
