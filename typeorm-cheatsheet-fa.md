# 📚 چک‌لیست TypeORM

**زبان‌های پشتیبانی‌شده:**
- [English](./README.md)



سلام دوستان! 😊

TypeORM یکی از بهترین ORM‌ها برای TypeScript و JavaScript است که از پایگاه‌داده‌های مختلفی مانند MySQL، PostgreSQL، SQLite و غیره پشتیبانی می‌کند. این ابزار فوق‌العاده به شما کمک می‌کند تا ارتباطات با پایگاه‌داده را ساده کنید و از قابلیت‌های TypeScript به بهترین شکل استفاده کنید.

---

## 🚀 1. نصب

برای شروع، اول از همه باید TypeORM و درایورهای پایگاه‌داده مورد نیاز را نصب کنید. با استفاده از این دستورات در ترمینال، کار را آغاز کنید:

```bash
npm install typeorm
npm install mysql2 # برای MySQL/MariaDB، برای پایگاه‌داده‌های دیگر تنظیمات را تغییر دهید
```

**گزینه‌های پایگاه‌داده دیگر**:
- PostgreSQL: `npm install pg`
- SQLite: `npm install sqlite3`
- MongoDB: `npm install mongodb`

---

## 🔗 2. اتصال به پایگاه‌داده

حالا بیایید یک **DataSource** برای اتصال به پایگاه‌داده‌مان بسازیم. یک فایل به نام `data-source.ts` ایجاد کنید و کد زیر را در آن قرار دهید:

```ts
import { DataSource } from "typeorm";

export const AppDataSource = new DataSource({
  type: "mysql", // یا نوع پایگاه‌داده دیگر: 'postgres'، 'sqlite' و غیره
  host: "localhost",
  port: 3306, // پورت MySQL، در صورت لزوم تغییر دهید
  username: "root",
  password: "password",
  database: "test_db",
  synchronize: true, // به‌طور خودکار مدل‌ها را به پایگاه‌داده همگام‌سازی کنید (در تولید غیرفعال شود)
  logging: false, // برای فعال‌سازی لاگ‌های پرس‌وجوی SQL
  entities: [__dirname + "/entities/*.ts"], // مسیر فایل‌های موجودیت شما
  migrations: [],
  subscribers: [],
});
```

حالا اتصال را راه‌اندازی کنیم:

```ts
AppDataSource.initialize()
  .then(() => {
    console.log("✅ اتصال به پایگاه‌داده با موفقیت انجام شد!");
  })
  .catch((err) => {
    console.error("❌ خطا در اتصال به پایگاه‌داده", err);
  });
```

🔗 **[اطلاعات بیشتر درباره گزینه‌های DataSource](https://typeorm.io/data-source-options)**

---

## 🏗 3. تعریف موجودیت‌ها

موجودیت‌ها نمایانگر جداول پایگاه‌داده هستند. بیایید یک موجودیت به نام `User.ts` بسازیم:

```ts
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number; // کلید اصلی خودکار تولیدشده

  @Column({ length: 100 })
  name: string; // ستون رشته‌ای با حداکثر طول 100

  @Column()
  age: number; // ستون عددی صحیح

  @Column({ unique: true })
  email: string; // ستون ایمیل یکتا

  @Column({ default: true })
  isActive: boolean; // بولین با مقدار پیش‌فرض
}
```

### 📑 نوع‌های ستون موجودیت
- `@PrimaryGeneratedColumn()` - کلید اصلی خودکار تولیدشده.
- `@Column({ type: "text" })` - برای فیلدهای متنی.
- `@Column({ type: "int" })` - برای فیلدهای عددی صحیح.
- `@Column({ type: "boolean" })` - برای مقادیر بولین.
- `@Column({ default: "value" })` - تعیین مقدار پیش‌فرض برای یک ستون.

🔗 **[کاوش در نوع‌ها و گزینه‌های موجودیت‌ها](https://typeorm.io/entities)**

---

## 🛠 4. مخزن (عملیات CRUD)

### 4.1 📥 یافتن داده‌ها

```ts
const userRepository = AppDataSource.getRepository(User);

// یافتن تمام کاربران
const users = await userRepository.find();

// یافتن یکی با ID
const user = await userRepository.findOneBy({ id: 1 });
```

### 4.2 ➕ وارد کردن داده‌ها

```ts
const newUser = userRepository.create({
  name: "Jane Doe",
  age: 28,
  email: "jane@example.com",
});

await userRepository.save(newUser);
```

### 4.3 🔄 به‌روزرسانی داده‌ها

```ts
const userToUpdate = await userRepository.findOneBy({ id: 1 });

if (userToUpdate) {
  userToUpdate.age = 30;
  await userRepository.save(userToUpdate);
}
```

### 4.4 ❌ حذف داده‌ها

```ts
const userToRemove = await userRepository.findOneBy({ id: 1 });

if (userToRemove) {
  await userRepository.remove(userToRemove);
}
```

🔗 **[اطلاعات بیشتر درباره مخازن](https://typeorm.io/working-with-repository)**

---

## 🔍 5. پرس‌وجوی پیشرفته

### 5.1 سازنده پرس‌وجو

برای پرس‌وجوهای پیچیده‌تر از سازنده پرس‌وجو استفاده کنید:

```ts
const users = await userRepository
  .createQueryBuilder("user")
  .where("user.age > :age", { age: 18 })
  .andWhere("user.isActive = :isActive", { isActive: true })
  .getMany();
```

### 5.2 صفحه‌بندی و مرتب‌سازی

```ts
const paginatedUsers = await userRepository.find({
  skip: 0, // جابه‌جایی
  take: 10, // محدودیت
  order: { age: "ASC" }, // مرتب‌سازی بر اساس سن به صورت صعودی
});
```

🔗 **[اطلاعات بیشتر درباره سازنده پرس‌وجو](https://typeorm.io/select-query-builder)**

---

## 🔄 6. روابط (همبستگی‌ها)

### 6.1 رابطه یک به چند

در موجودیت `User.ts`:

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

در موجودیت `Post.ts`:

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

### 6.2 بارگذاری اشتیاقی و تنبل

- **بارگذاری اشتیاقی**: موجودیت‌های مرتبط به‌طور خودکار بارگذاری می‌شوند.
- **بارگذاری تنبل**: موجودیت‌های مرتبط فقط زمانی بارگذاری می‌شوند که به آن‌ها دسترسی پیدا شود.

```ts
@Entity()
export class User {
  @OneToMany(() => Post, (post) => post.user, { eager: true }) // بارگذاری اشتیاقی
  posts: Post[];
}
```

🔗 **[اطلاعات بیشتر درباره روابط](https://typeorm.io/relations)**

---

## 📅 7. مهاجرت‌ها

### 7.1 تولید مهاجرت‌ها

```bash
typeorm migration:generate -n MigrationName
```

### 7.2 اجرای مهاجرت‌ها

```bash
typeorm migration:run
```

### 7.3 برگرداندن مهاجرت‌ها

```bash
typeorm migration:revert
```

### مثال مهاجرت

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

🔗 **[اطلاعات بیشتر درباره مهاجرت‌ها](https://typeorm.io/migrations)**

---

## 🧰 8. دکوراتورهای متداول

- `@Entity()` – تعریف یک جدول.
- `@PrimaryGeneratedColumn()` – کلید اصلی خودکار تولیدشده.
- `@Column()` – تعریف یک ستون با نوع آن.
- `@OneToMany()` / `@ManyToOne()` – تعریف روابط یک به چند.
- `@JoinColumn()` – برای مشخص کردن کلید خارجی.

🔗 **[کاوش در دکوراتورهای TypeORM](https://typeorm.io/decorators)**

---

## 📖 منابع مفید

- [مستندات TypeORM](https://typeorm.io/)
- [گیت‌هاب TypeORM](https://github.com/typeorm/typeorm)
- [آموزش TypeORM](https://wanago.io/2020/03/23/typescript-typeorm-tutorial/)

---

این چک‌لیست یک مرور کلی از امکانات TypeORM را ارائه می‌دهد. برای اطلاعات دقیق‌تر، به مستندات مراجعه کنید. موفق باشید! 🚀
