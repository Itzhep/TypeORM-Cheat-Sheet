# 📚 TypeORM Cheat Sheet

**Supported Languages:**

- [فارسی](./typeorm-cheatsheet-fa.md)

TypeORM is an ORM for TypeScript and JavaScript that supports various databases such as MySQL, PostgreSQL, SQLite, and more. It simplifies database interactions and provides strong TypeScript support.

---

## 🚀 1. Installation

First, install TypeORM and the required database drivers.

```bash
npm install typeorm
npm install mysql2 # For MySQL/MariaDB, adjust for other databases
```

**Other database options**:
- PostgreSQL: `npm install pg`
- SQLite: `npm install sqlite3`
- MongoDB: `npm install mongodb`

---

## 🔗 2. Connecting to the Database

Set up a **DataSource** to connect to your database in `data-source.ts`:

```ts
import { DataSource } from "typeorm";

export const AppDataSource = new DataSource({
  type: "mysql", // Or another database type: 'postgres', 'sqlite', etc.
  host: "localhost",
  port: 3306, // MySQL port, change if necessary
  username: "root",
  password: "password",
  database: "test_db",
  synchronize: true, // Automatically sync models to database (disable in production)
  logging: false, // Enable for SQL query logs
  entities: [__dirname + "/entities/*.ts"], // Path to your entity files
  migrations: [],
  subscribers: [],
});
```

Initialize the connection:

```ts
AppDataSource.initialize()
  .then(() => {
    console.log("✅ Database connected successfully!");
  })
  .catch((err) => {
    console.error("❌ Error connecting to the database", err);
  });
```

🔗 **[Learn more about DataSource options](https://typeorm.io/data-source-options)**

---

## 🏗 3. Defining Entities

Entities represent database tables. Create an entity like `User.ts`:

```ts
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number; // Auto-generated primary key

  @Column({ length: 100 })
  name: string; // String column with a max length of 100

  @Column()
  age: number; // Integer column

  @Column({ unique: true })
  email: string; // Unique email column

  @Column({ default: true })
  isActive: boolean; // Boolean with default value
}
```

### 📑 Entity Column Types
- `@PrimaryGeneratedColumn()` - Auto-increment primary key.
- `@Column({ type: "text" })` - For text fields.
- `@Column({ type: "int" })` - For integer fields.
- `@Column({ type: "boolean" })` - For boolean values.
- `@Column({ default: "value" })` - Set default value for a column.

🔗 **[Explore entity types and options](https://typeorm.io/entities)**

---

## 🛠 4. Repository (CRUD Operations)

### 4.1 📥 Find Data

```ts
const userRepository = AppDataSource.getRepository(User);

// Find all users
const users = await userRepository.find();

// Find one by ID
const user = await userRepository.findOneBy({ id: 1 });
```

### 4.2 ➕ Insert Data

```ts
const newUser = userRepository.create({
  name: "Jane Doe",
  age: 28,
  email: "jane@example.com",
});

await userRepository.save(newUser);
```

### 4.3 🔄 Update Data

```ts
const userToUpdate = await userRepository.findOneBy({ id: 1 });

if (userToUpdate) {
  userToUpdate.age = 30;
  await userRepository.save(userToUpdate);
}
```

### 4.4 ❌ Delete Data

```ts
const userToRemove = await userRepository.findOneBy({ id: 1 });

if (userToRemove) {
  await userRepository.remove(userToRemove);
}
```

🔗 **[Learn more about Repositories](https://typeorm.io/working-with-repository)**

---

## 🔍 5. Advanced Queries

### 5.1 Query Builder

Use the query builder for more complex queries:

```ts
const users = await userRepository
  .createQueryBuilder("user")
  .where("user.age > :age", { age: 18 })
  .andWhere("user.isActive = :isActive", { isActive: true })
  .getMany();
```

### 5.2 Pagination & Sorting

```ts
const paginatedUsers = await userRepository.find({
  skip: 0, // Offset
  take: 10, // Limit
  order: { age: "ASC" }, // Order by age ascending
});
```

🔗 **[Learn more about Query Builder](https://typeorm.io/select-query-builder)**

---

## 🔄 6. Relations (Associations)

### 6.1 One-to-Many Relation

In the `User.ts` entity:

```ts
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from "typeorm";
import { Post } from "./Post";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Post, (post) => post.user)
  posts: Post[];
}
```

In the `Post.ts` entity:

```ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from "typeorm";
import { User } from "./User";

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, (user) => user.posts)
  user: User;
}
```

### 6.2 Eager vs Lazy Loading

- **Eager Loading**: Load related entities automatically.
- **Lazy Loading**: Load related entities only when accessed.

```ts
@Entity()
export class User {
  @OneToMany(() => Post, (post) => post.user, { eager: true }) // Eager loading
  posts: Post[];
}
```

🔗 **[Learn more about relations](https://typeorm.io/relations)**

---

## 📅 7. Migrations

### 7.1 Generating Migrations

```bash
typeorm migration:generate -n MigrationName
```

### 7.2 Running Migrations

```bash
typeorm migration:run
```

### 7.3 Reverting Migrations

```bash
typeorm migration:revert
```

### Migration Example

```ts
import { MigrationInterface, QueryRunner } from "typeorm";

export class UserTableMigration1673423701439 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      CREATE TABLE user (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(100),
        email VARCHAR(100) UNIQUE
      )
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP TABLE user`);
  }
}
```

🔗 **[Learn more about migrations](https://typeorm.io/migrations)**

---

## 🧰 8. Common Decorators

- `@Entity()` – Defines a table.
- `@PrimaryGeneratedColumn()` – Auto-increment primary key.
- `@Column()` – Defines a column with its type.
- `@OneToMany()` / `@ManyToOne()` – Defines one-to-many relationships.
- `@JoinColumn()` – Explicitly specifies which column the relation should use.
- `@CreateDateColumn()` / `@UpdateDateColumn()` – Automatically manage `createdAt` and `updatedAt` timestamps.

---

## 🌍 9. Additional Links

- 📘 [Official TypeORM Documentation](https://typeorm.io)
- 📺 [TypeORM Tutorial for Beginners (YouTube)](https://www.youtube.com/watch?v=JaTbzPcyiOE&pp=ygUIdHlwZW9ybSA%3D)
- 📚 [TypeORM GitHub Repo](https://github.com/typeorm/typeorm)

---

### 📝 Pro Tips:
1. **Disable `synchronize` in production**: Automatically syncing models to the database is handy during development, but risky in production. Use migrations instead!
2. **Use `logging: true` in development** to track SQL queries executed by TypeORM.
3. **Use transactions** for complex operations to ensure consistency. Example: `await connection.transaction(async manager => { ... })`.

---

This cheat sheet should give you a solid starting point for building applications with TypeORM. Happy coding! 🎉
