---
title: 'Database operations'
date: 2024-11-28T11:59:08+10:00
draft: false
---

# Database operations

## Overview
Common database operations include Create, Read, Update, and Delete (often referred to as CRUD). These operations are essential for managing data within a database.

### 1. Create
- **Description**: Adding new records to a database.
- **How it's done**: Using SQL `INSERT` statements or equivalent methods in NoSQL databases.
- **Next.js Example**: Use an API route to handle the creation of new records.

### 2. Read
- **Description**: Retrieving data from the database.
- **How it's done**: Using SQL `SELECT` statements or querying methods in NoSQL databases.
- **Next.js Example**: Fetch data in `getServerSideProps` or `getStaticProps` for server-side rendering.

### 3. Update
- **Description**: Modifying existing records in the database.
- **How it's done**: Using SQL `UPDATE` statements or update methods in NoSQL databases.
- **Next.js Example**: Use an API route to handle updates based on user input.

### 4. Delete
- **Description**: Removing records from the database.
- **How it's done**: Using SQL `DELETE` statements or delete methods in NoSQL databases.
- **Next.js Example**: Use an API route to handle deletions.

## Example of Using Database Operations in a Next.js Function

Here’s a simplified example of how you might implement these operations in a Next.js API route:

```ts
// app/api/users/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  const users = await db.user.findMany();
  return NextResponse.json(users);
}

export async function POST(req: Request) {
  const { name, email } = await req.json();
  const user = await db.user.create({ data: { name, email } });
  return NextResponse.json(user);
}

export async function PUT(req: Request) {
  const { id, name, email } = await req.json();
  const user = await db.user.update({ where: { id }, data: { name, email } });
  return NextResponse.json(user);
}

export async function DELETE(req: Request) {
  const { id } = await req.json();
  await db.user.delete({ where: { id } });
  return NextResponse.json({ message: 'User deleted' });
}
```

## Explanation of the Example
- **POST**: Creates a new user by calling the `create` method on the database.
- **GET**: Reads all users using the `findMany` method.
- **PUT**: Updates an existing user by calling the `update` method with the user's ID and new data.
- **DELETE**: Deletes a user based on the provided ID using the `delete` method.

This structure lets you do common database operations, here done in Next.js using the App Router and TypeScript.
