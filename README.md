# BRPC - A Type-Safe, Flexible Router for Bun

**brpc** is a minimal yet powerful router and server framework for Bun, inspired by tRPC but tailored for high performance and flexibility. It offers:

✅ **End-to-end type safety** with TypeScript and Zod validation  
✅ **Procedures with built-in middleware** for structured route handling  
✅ **WebSocket subscriptions & real-time updates**  
✅ **Automatic handling of file uploads and form-data**  
✅ **Global middlewares for rate-limiting, path blocking, and more**  
✅ **Streaming support for queries, mutations, and form mutations**

---

## 🚀 Getting Started

### 1️⃣ Create a Router Instance

```ts
import { createRouter } from "brpc";
import type { BaseContext } from "./types";

// Define custom context type
export interface CustomContext extends BaseContext {
  user?: any;
}

const router = createRouter({
  context: async () => ({} as CustomContext),
  routes: routes,
  globalMiddlewares: [],
  prefix: "/",
});

// Start the server
router.listen(3000, () => {
  console.log("🚀 brpc server running on http://localhost:3000");
});
```

### 2️⃣ Define Procedures
Procedures allow structured route handling with middleware, similar to tRPC.

```ts
import { createProcedure } from "brpc";

// Base procedure
export const procedure = createProcedure<CustomContext>();

// Middleware-protected procedure
export const userProcedure = procedure.use(async (ctx) => {
  if (!ctx.user) throw new Error("Unauthorized");
});
```

Middleware allows you to attach logic to a procedure, ensuring structured validation and security.

---

## 📌 Defining Routes

### Basic Query Procedure
```ts
const routes = {
  index: procedure.query(async ({ ctx }) => {
    return "Hello from brpc";
  }),
};
```
➡️ This will return `"Hello from brpc"` when a `GET` request is made to `/`.

### Serving Static Files
```ts
const routes = {
  app: procedure.file(async () => Bun.file(".app/index.html")),
  "client.js": procedure.file(async () => Bun.file(".app/client.js")),
};
```
➡️ Serve an HTML app and client-side JS files easily. brpc automatically sets appropriate headers.

---

## 🔥 Real-Time Subscriptions & Streaming

brpc natively supports WebSocket subscriptions. Additionally, queries, mutations, and form mutations now support **streaming responses**, allowing real-time updates directly from server-side procedures.

```ts
const routes = {
  getMessages: procedure.query(async ({ ctx, stream }) => {
    for await (const message of messageStream()) {
      stream.status({ event: "newMessage", data: message });
    }
  }),

  sendMessage: procedure
    .input(z.object({ username: z.string(), text: z.string() }))
    .mutation(async ({ ctx, input, stream }) => {
      const newMessage = await pushMessage(input);
      stream.status({ event: "messageSent", data: newMessage });
      return newMessage;
    }),

  ":channelId": procedure
    .input(
      z.object({
        username: z.string(),
        text: z.string(),
        pushed: z.boolean().nullish(),
      })
    )
    .subscription(async ({ ctx, input }) => {
      if (input.text === "withError") throw new Error("Message with error");
      if (!input.pushed) await pushMessage(input);
      return { ...input, timestamp: Date.now() };
    }),
};
```
✅ **Subscriptions are internal** and automatically push messages to connected clients.  
✅ **Queries, mutations, and form mutations can stream data** in real-time using `stream.status`.  
✅ **Errors thrown in subscriptions** will propagate to listeners, allowing UI rollback.  

---

## 📂 Handling File Uploads (Multipart/Form-Data)

brpc makes file uploads seamless, automatically parsing form-data into structured input:

```ts
const routes = {
  withFormData: procedure
    .input(
      z.object({
        files: createFileSchema({
          acceptedTypes: { audio: "*" },
          maxSize: 5,
        }).array(),
      })
    )
    .formMutation(async ({ ctx, input, stream }) => {
      console.log(input.files);

      // Example: Upload to S3
      const command = new PutObjectCommand({
        Bucket: process.env.AWS_BUCKET_NAME!,
      });

      // Track upload progress
      stream.status({ event: "upload", progress: 50, status: "uploading" });
      
      return { success: true };
    }),
};
```
✅ **Automatic file validation** with Zod  
✅ **Real-time upload progress tracking** via the `stream.status` helper  

---

## 🌍 Built-in Features

| Feature | Description |
|---------|------------|
| ✅ **Middleware Support** | Global and per-procedure middleware for security, validation, and more |
| ✅ **WebSockets & Subscriptions** | Real-time messaging and internal-only WebSocket topics |
| ✅ **Streaming for Queries & Mutations** | Live data updates for queries, mutations, and form mutations |
| ✅ **Static File Serving** | Serve HTML, JS, and other static assets effortlessly |
| ✅ **Multipart/Form-Data Handling** | File uploads with validation, streaming, and real-time progress tracking |
| ✅ **Error Handling** | Automatic handling of internal, context, and middleware errors |
| ✅ **Rate Limiting & Security** | Global middleware like rate limiting and path blocking |

---

## 🚀 Future Plans
✅ **JSX Rendering Support** (server-side rendering with Bun)  
✅ **Extended CORS Configuration**  
✅ **Server-Sent Events (SSE) for streaming responses**  
✅ **Advanced authentication middleware** (JWT, OAuth, etc.)  
✅ **Edge compatibility for ultra-fast execution**

📌 **brpc is already powering production apps!** It’s built for speed, flexibility, and maintainability. 🎯

---

## 🤝 Get Involved
Interested in contributing or using brpc? Let’s build something great together! 🚀

