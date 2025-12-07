# @nexus/core

**Enterprise-grade infrastructure library for Node.js backend development**

Nexus.js Core provides Go-style context propagation, enterprise-level architecture patterns, and high-performance caching for building scalable Node.js applications.

## 🎯 Vision

Bring **Vercel-grade Developer Experience (DX)** and **Enterprise-grade stability** to Node.js backend development.

## ✨ Features

### 🔄 Context Propagation (Go-inspired)
- **AsyncLocalStorage-based**: Automatic context propagation across async boundaries
- **No manual passing**: Context flows through your call stack automatically
- **Type-safe**: Full TypeScript support with generics
- **Cancellation tokens**: Built-in support for request cancellation and resource cleanup

### 🚀 High-Performance Caching
- **Powered by Aethel.TS**: Leverages optimized LRUCache implementation
- **L1 in-memory caching**: Perfect for auth tokens, permissions, and frequently accessed data
- **TTL support**: Automatic expiration of cached entries
- **getOrSet pattern**: Simplified cache-aside pattern implementation

### 🏗️ Enterprise Architecture
- **Framework-agnostic core**: Works with Express, Fastify, Koa, or pure Node.js
- **Hexagonal architecture ready**: Clear separation between ports and adapters
- **Pluggable storage**: Interface-based design for easy adapter implementation

## 📦 Installation

```bash
npm install @nexus/core
```

## 🚀 Quick Start

### Context Propagation

```typescript
import { RequestContext } from '@nexus/core';

// In your request handler
app.use((req, res, next) => {
  RequestContext.run(
    {
      traceId: generateTraceId(),
      requestId: req.id,
      userId: req.user?.id,
      timestamp: Date.now()
    },
    async () => {
      await next();
    }
  );
});

// Anywhere in your application
async function serviceLayer() {
  const context = RequestContext.current();
  const traceId = context?.get('traceId');
  
  console.log(`Processing request ${traceId}`);
  
  // Context automatically propagates through async calls
  await dataLayer();
}

async function dataLayer() {
  const context = RequestContext.current();
  const userId = context?.get('userId');
  
  // No need to pass context manually!
  console.log(`Querying data for user ${userId}`);
}
```

### Cancellation Tokens

```typescript
import { RequestContext } from '@nexus/core';

async function longRunningOperation() {
  const context = RequestContext.current();
  
  // Register cleanup handler
  context?.onCancel(() => {
    console.log('Cleaning up resources...');
    // Close connections, release locks, etc.
  });
  
  // Check cancellation status periodically
  while (!context?.isCancelled()) {
    await processChunk();
  }
}

// Trigger cancellation
setTimeout(() => {
  context.cancel(); // All registered callbacks will execute
}, 5000);
```

### Cache Management

```typescript
import { CacheManager } from '@nexus/core';

const userCache = new CacheManager<string, User>(1000);

async function getUser(userId: string): Promise<User> {
  // Cache-aside pattern made easy
  return userCache.getOrSet(
    userId,
    async () => {
      // This only runs on cache miss
      return await db.users.findById(userId);
    },
    60000 // TTL: 60 seconds
  );
}

// First call: hits database
const user1 = await getUser('user-123'); // 200ms (DB latency)

// Second call: hits cache
const user2 = await getUser('user-123'); // <1ms (cache hit)
```

## 🧪 Running POC Demonstrations

This package includes three comprehensive POC (Proof of Concept) demonstrations:

### 1. Context Propagation POC
```bash
npm run poc
```

Demonstrates:
- ✅ Context flowing through multiple async layers
- ✅ No manual context passing required
- ✅ Concurrent requests maintaining isolated contexts
- ✅ Promise.all and parallel operations with context preservation

### 2. Cancellation Token POC
```bash
npm run poc:cancellation
```

Demonstrates:
- ✅ Graceful cancellation of long-running operations
- ✅ Resource cleanup on cancellation
- ✅ Multiple cleanup callbacks
- ✅ Immediate callback execution for late registrations

### 3. Cache Manager POC
```bash
npm run poc:cache
```

Demonstrates:
- ✅ Basic cache operations (get/set/delete)
- ✅ User data caching pattern
- ✅ Permission caching with TTL
- ✅ LRU eviction behavior
- ✅ Performance comparison (cache vs. database)

## 🏛️ Architecture

### Context Flow

```
HTTP Request
    ↓
RequestContext.run({traceId, userId, ...})
    ↓
[Your Application Code]
    ↓
Service Layer (auto-access to context)
    ↓
Data Layer (auto-access to context)
    ↓
Database/External APIs (auto-access to context)
```

### Key Components

- **IContext**: Generic interface for context operations
- **RequestContext**: AsyncLocalStorage-based implementation
- **CacheManager**: High-performance caching layer
- **NexusContextData**: Standard context data structure

## 🎓 Design Philosophy

### 1. **Go-inspired Context Propagation**
Node.js lacks built-in request-scoped context. We solve this using AsyncLocalStorage, similar to Go's context package.

### 2. **C#-inspired Architecture**
Clean separation of concerns, interface-based design, and type safety inspired by C# Identity system.

### 3. **Performance-First**
Integration with Aethel.TS ensures enterprise-grade performance for caching and data structures.

### 4. **Developer Experience**
Zero-config defaults, intuitive APIs, and comprehensive TypeScript support.

## 📊 Performance

Based on our POC testing:

- **Context Access**: <0.001ms per operation
- **Cache Hit**: ~1000x faster than database calls
- **Memory Overhead**: Minimal (AsyncLocalStorage is optimized)
- **No Callback Hell**: Clean async/await patterns throughout

## 🗺️ Roadmap

- [x] Core context propagation system
- [x] Cancellation token support
- [x] Aethel.TS cache integration
- [ ] Express adapter (`@nexus/adapter-express`)
- [ ] Auth system (`@nexus/auth`)
- [ ] gRPC adapter (`@nexus/adapter-grpc`)
- [ ] CLI tooling (`@nexus/cli`)
- [ ] Database adapters (`@nexus/adapter-pg`, `@nexus/adapter-mongo`)

## 🤝 Contributing

This is a POC project. Feedback and contributions welcome!

## 📄 License

MIT

## 🔗 Related Projects

- **Aethel.TS**: High-performance data structures (used internally)
- **@nexus/auth**: C# Identity-inspired auth system (coming soon)
- **@nexus/cli**: Developer experience tooling (coming soon)

---

**Built with ❤️ for enterprise Node.js development**