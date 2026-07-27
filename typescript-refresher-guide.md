# TypeScript Refresher — The 80% You'll Actually Use

Aimed at Next.js (frontend) + Node.js (backend). Every concept has a runnable-style example.

---

## 1. Basic Types

```ts
let username: string = "Faisal";
let age: number = 25;
let isActive: boolean = true;
let tags: string[] = ["dev", "react"];
let coords: [number, number] = [23.8, 90.4]; // tuple - fixed length & types

// TypeScript usually infers types, so this is often unnecessary:
let city = "Dhaka"; // inferred as string
```

**Rule of thumb:** let inference work. Only annotate when TS can't infer (function params, empty arrays, etc.).

---

## 2. `interface` vs `type`

Both describe object shapes. 90% of the time they're interchangeable — pick one convention and stick with it.

```ts
interface User {
  id: string;
  name: string;
  email?: string;       // optional property
  readonly createdAt: Date; // can't be reassigned after creation
}

type Product = {
  id: string;
  price: number;
};
```

**When they differ:**
- `interface` can be re-opened/extended later (declaration merging) — good for library/API shapes.
- `type` can represent unions, intersections, primitives — `interface` cannot.

```ts
type Status = "pending" | "active" | "banned"; // only `type` can do this
type ID = string | number;
```

**Extending:**
```ts
interface Admin extends User {
  role: "admin";
}

type AdminType = User & { role: "admin" }; // intersection = same effect
```

---

## 3. Union, Intersection & Narrowing

```ts
function printId(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase()); // TS knows it's a string here
  } else {
    console.log(id.toFixed(2));    // TS knows it's a number here
  }
}
```

**Discriminated unions** — the pattern you'll use constantly for API responses, form states, Redux actions:

```ts
type ApiResponse<T> =
  | { status: "success"; data: T }
  | { status: "error"; message: string };

function handle(res: ApiResponse<User>) {
  if (res.status === "success") {
    console.log(res.data); // TS narrows to the success branch
  } else {
    console.log(res.message);
  }
}
```

---

## 4. Functions

```ts
// params, optional param, default value, return type
function createUser(name: string, age?: number, role: string = "user"): User {
  return { id: crypto.randomUUID(), name, createdAt: new Date() };
}

// rest params
function sum(...nums: number[]): number {
  return nums.reduce((a, b) => a + b, 0);
}

// arrow function with typed params
const multiply = (a: number, b: number): number => a * b;

// function type as a variable
let handler: (err: Error | null, data?: string) => void;
```

---

## 5. Arrays, Objects & `Record`

```ts
const ids: number[] = [1, 2, 3];
const users: User[] = [];

// Record<KeyType, ValueType> — very common for lookup maps
const rolePermissions: Record<"admin" | "editor" | "viewer", string[]> = {
  admin: ["read", "write", "delete"],
  editor: ["read", "write"],
  viewer: ["read"],
};
```

---

## 6. Enums & Literal Types

```ts
// Prefer union literals over enums in modern TS/Next.js codebases:
type OrderStatus = "PLACED" | "SHIPPED" | "DELIVERED";

// enums still show up in some backends:
enum Role {
  Admin = "ADMIN",
  User = "USER",
}
```

---

## 7. Generics

The most important intermediate concept — used everywhere in React props, API utils, and DB queries.

```ts
function wrapInArray<T>(item: T): T[] {
  return [item];
}
wrapInArray<string>("hi"); // ["hi"]
wrapInArray(5);            // inferred as number[]

// Generic interface — perfect for API wrappers
interface ApiResult<T> {
  data: T;
  error: string | null;
}

function fetchUser(): ApiResult<User> {
  return { data: { id: "1", name: "Faisal", createdAt: new Date() }, error: null };
}

// Generic constraints
function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

---

## 8. Utility Types (huge time-saver, use constantly)

```ts
interface Todo {
  id: string;
  title: string;
  done: boolean;
}

type PartialTodo = Partial<Todo>;      // all props optional (good for PATCH/update DTOs)
type TodoPreview = Pick<Todo, "id" | "title">; // subset of properties
type TodoNoId = Omit<Todo, "id">;      // everything except one prop (good for "create" payloads)
type RequiredTodo = Required<PartialTodo>; // opposite of Partial
type ReadonlyTodo = Readonly<Todo>;    // immutable object

// Extracting types from existing functions/values — huge for Next.js
function getTodo() {
  return { id: "1", title: "Learn TS", done: false };
}
type Todo2 = ReturnType<typeof getTodo>;

function saveTodo(todo: Todo) {}
type SaveTodoParams = Parameters<typeof saveTodo>; // [Todo]
```

---

## 9. `unknown` vs `any` vs Type Assertions

```ts
let a: any = 5;       // avoid — disables type checking entirely
let b: unknown = 5;   // safer "any" — must narrow before use

function process(val: unknown) {
  if (typeof val === "string") {
    console.log(val.toUpperCase()); // ok, narrowed
  }
}

// Type assertion — "trust me, I know the type" (use sparingly)
const input = document.getElementById("email") as HTMLInputElement;
const value = input.value;
```

Use `unknown` for untyped external data (API responses, `JSON.parse`) instead of `any`.

---

## 10. Classes (backend-relevant)

```ts
class UserService {
  private db: Map<string, User> = new Map();

  constructor(private readonly logger: Console = console) {}

  public getUser(id: string): User | undefined {
    return this.db.get(id);
  }
}

abstract class Repository<T> {
  abstract findById(id: string): T | undefined;
}
```

---

## 11. Async / Promises

```ts
async function getUserById(id: string): Promise<User | null> {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) return null;
  const data: User = await res.json();
  return data;
}
```

`JSON.parse`/`res.json()` return `any` by default — annotate the variable so you get safety back.

---

## 12. Next.js — Common TypeScript Patterns

### Typing page/component props (App Router)

```tsx
// app/users/[id]/page.tsx
interface PageProps {
  params: { id: string };
  searchParams?: { [key: string]: string | string[] | undefined };
}

export default function UserPage({ params }: PageProps) {
  return <div>User: {params.id}</div>;
}
```

### Typing a component with children

```tsx
interface CardProps {
  title: string;
  children: React.ReactNode;
}

function Card({ title, children }: CardProps) {
  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  );
}
```

### Route Handlers (App Router API routes)

```ts
// app/api/users/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function GET(req: NextRequest) {
  return NextResponse.json({ users: [] });
}

export async function POST(req: NextRequest) {
  const body: { name: string } = await req.json();
  return NextResponse.json({ id: "1", name: body.name });
}
```

### Server Actions

```ts
"use server";

interface CreatePostInput {
  title: string;
  content: string;
}

export async function createPost(input: CreatePostInput) {
  // db call here
  return { success: true };
}
```

### Typing hooks

```ts
const [user, setUser] = useState<User | null>(null);
const ref = useRef<HTMLInputElement>(null);
```

---

## 13. Node.js / Express — Common TypeScript Patterns

```ts
import express, { Request, Response, NextFunction } from "express";

const app = express();

interface AuthRequest extends Request {
  user?: { id: string; role: string };
}

app.get("/users/:id", (req: Request, res: Response) => {
  const { id } = req.params; // typed as string
  res.json({ id });
});

// typed middleware
function authMiddleware(req: AuthRequest, res: Response, next: NextFunction) {
  req.user = { id: "1", role: "admin" };
  next();
}

// typed request body (use a generic on Request)
app.post("/users", (req: Request<{}, {}, { name: string; email: string }>, res: Response) => {
  const { name, email } = req.body; // fully typed
  res.status(201).json({ name, email });
});
```

**Typing environment variables:**
```ts
declare global {
  namespace NodeJS {
    interface ProcessEnv {
      DATABASE_URL: string;
      JWT_SECRET: string;
    }
  }
}
```

---

## 14. `tsconfig.json` essentials

```json
{
  "compilerOptions": {
    "strict": true,           // always keep this on
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "noUncheckedIndexedAccess": true // catches array[i] possibly undefined
  }
}
```

`strict: true` is non-negotiable — it enables `strictNullChecks`, which is where most real-world type safety comes from.

---

## 15. Quick Reference — things you'll type constantly

| Need | Syntax |
|---|---|
| Optional prop | `name?: string` |
| Nullable | `name: string \| null` |
| Non-null assertion (careful!) | `user!.name` |
| Optional chaining | `user?.profile?.avatar` |
| Array of objects | `User[]` or `Array<User>` |
| Function type | `(x: number) => void` |
| Keyof | `keyof User` → `"id" \| "name" \| ...` |
| Index signature | `{ [key: string]: number }` |

---

## What to practice next
1. Rebuild one of your existing components (e.g. from BhumiBazar or LingoLink) with strict typed props.
2. Type one full Next.js route handler + the fetch call that consumes it, end-to-end.
3. Write one small Express CRUD route with fully typed `req.body`/`req.params`.

That loop — type the frontend call, type the backend handler, make sure they agree — is basically 80% of real-world TS work.
