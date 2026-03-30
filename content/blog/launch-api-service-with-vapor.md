---
slug: "launch-api-service-with-vapor"
title: "Launch a Small API Service Using Vapor"
date: "2026-03-22"
author: "Abdullah Alhaider"
github: "cs4alhaider"
twitter: "cs4alhaider"
tags: ["vapor", "server-side swift", "api", "backend", "tutorial"]
excerpt: "Learn how to build and launch a fully working REST API using Vapor — the popular server-side Swift framework. We'll build a simple task manager API from scratch."
---

## Why Vapor?

If you're a Swift developer, you already know the language. Why learn another one just to build an API? **Vapor** lets you use Swift on the server — same type safety, same concurrency model, same ecosystem. It's fast, expressive, and feels right at home if you've been writing SwiftUI or UIKit.

Vapor gives you:

- **Routing** with type-safe parameters
- **Codable** integration for JSON out of the box
- **Fluent ORM** for database access (PostgreSQL, MySQL, SQLite, MongoDB)
- **async/await** support built on SwiftNIO
- **Middleware**, authentication, WebSockets, and more

## Prerequisites

- macOS with Xcode installed (Swift 5.9 or greater — required by Vapor 4)
- Basic Swift knowledge
- Terminal access
- Homebrew (for installing the Vapor toolbox)

## Step 1: Install Vapor Toolbox

The Vapor toolbox CLI helps you create and manage projects:

```bash
brew install vapor
```

Verify the installation:

```bash
vapor --help
```

## Step 2: Create a New Project

```bash
vapor new TaskAPI -n
cd TaskAPI
```

The `-n` flag provides a bare bones template by automatically answering no to all questions. Open the project:

```bash
open Package.swift
```

This opens the project in Xcode. Wait for Swift Package Manager to resolve dependencies.

## Step 3: Understanding the Project Structure

Vapor projects follow a clean structure:

```
TaskAPI/
  Public/                 # Static files (if FileMiddleware is enabled)
  Sources/
    App/
      Controllers/        # Route handlers
      Migrations/          # Database schema changes
      Models/              # Fluent models
      configure.swift      # Registers services, routes, databases
      routes.swift         # Route definitions
      entrypoint.swift     # @main entry point
  Tests/
    AppTests/
  Package.swift
```

## Step 4: Define the Task Model

Create a `Task` model using Fluent. This maps to a database table.

Replace or create `Sources/App/Models/Task.swift`:

```swift
import Fluent
import Vapor

final class TaskItem: Model, Content {
    static let schema = "tasks"

    @ID(key: .id)
    var id: UUID?

    @Field(key: "title")
    var title: String

    @Field(key: "is_completed")
    var isCompleted: Bool

    @Timestamp(key: "created_at", on: .create)
    var createdAt: Date?

    init() {}

    init(id: UUID? = nil, title: String, isCompleted: Bool = false) {
        self.id = id
        self.title = title
        self.isCompleted = isCompleted
    }
}
```

We name it `TaskItem` to avoid conflicts with Swift's built-in `Task` type.

## Step 5: Create the Database Migration

Create `Sources/App/Migrations/CreateTask.swift`:

```swift
import Fluent

struct CreateTask: AsyncMigration {
    func prepare(on database: Database) async throws {
        try await database.schema("tasks")
            .id()
            .field("title", .string, .required)
            .field("is_completed", .bool, .required, .custom("DEFAULT FALSE"))
            .field("created_at", .datetime)
            .create()
    }

    func revert(on database: Database) async throws {
        try await database.schema("tasks").delete()
    }
}
```

## Step 6: Build the Controller

Create `Sources/App/Controllers/TaskController.swift` with full CRUD operations:

```swift
import Fluent
import Vapor

struct TaskController: RouteCollection {
    func boot(routes: RoutesBuilder) throws {
        let tasks = routes.grouped("api", "tasks")
        tasks.get(use: index)
        tasks.post(use: create)
        tasks.group(":taskID") { task in
            task.get(use: show)
            task.put(use: update)
            task.delete(use: delete)
        }
    }

    // GET /api/tasks
    @Sendable
    func index(req: Request) async throws -> [TaskItem] {
        try await TaskItem.query(on: req.db).all()
    }

    // POST /api/tasks
    @Sendable
    func create(req: Request) async throws -> TaskItem {
        let task = try req.content.decode(TaskItem.self)
        try await task.save(on: req.db)
        return task
    }

    // GET /api/tasks/:taskID
    @Sendable
    func show(req: Request) async throws -> TaskItem {
        guard let task = try await TaskItem.find(
            req.parameters.get("taskID"), on: req.db
        ) else {
            throw Abort(.notFound)
        }
        return task
    }

    // PUT /api/tasks/:taskID
    @Sendable
    func update(req: Request) async throws -> TaskItem {
        guard let task = try await TaskItem.find(
            req.parameters.get("taskID"), on: req.db
        ) else {
            throw Abort(.notFound)
        }
        let updated = try req.content.decode(TaskItem.self)
        task.title = updated.title
        task.isCompleted = updated.isCompleted
        try await task.save(on: req.db)
        return task
    }

    // DELETE /api/tasks/:taskID
    @Sendable
    func delete(req: Request) async throws -> HTTPStatus {
        guard let task = try await TaskItem.find(
            req.parameters.get("taskID"), on: req.db
        ) else {
            throw Abort(.notFound)
        }
        try await task.delete(on: req.db)
        return .noContent
    }
}
```

## Step 7: Configure the App

Update `Sources/App/configure.swift` to register the database and migration:

```swift
import Fluent
import FluentSQLiteDriver
import Vapor

public func configure(_ app: Application) async throws {
    // Use SQLite for simplicity (in-memory for dev)
    app.databases.use(.sqlite(.memory), as: .sqlite)

    // Register migrations
    app.migrations.add(CreateTask())

    // Run migrations automatically
    try await app.autoMigrate()

    // Register routes
    try routes(app)
}
```

We're using SQLite in-memory for this tutorial. For production, swap to PostgreSQL:

```swift
app.databases.use(
    .postgres(hostname: "localhost", username: "vapor", password: "secret", database: "taskapi"),
    as: .psql
)
```

## Step 8: Register the Routes

Update `Sources/App/routes.swift`:

```swift
import Fluent
import Vapor

func routes(_ app: Application) throws {
    // Health check
    app.get("health") { req in
        ["status": "ok"]
    }

    // Register the task controller
    try app.register(collection: TaskController())
}
```

## Step 9: Run the Server

```bash
swift run App
```

You should see:

```
[ INFO ] Server starting on http://127.0.0.1:8080
```

## Step 10: Test Your API

Create a task:

```bash
curl -X POST http://127.0.0.1:8080/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Learn Vapor"}'
```

Response:

```json
{
  "id": "A1B2C3D4-...",
  "title": "Learn Vapor",
  "is_completed": false,
  "created_at": "2026-03-22T10:00:00Z"
}
```

List all tasks:

```bash
curl http://127.0.0.1:8080/api/tasks
```

Update a task:

```bash
curl -X PUT http://127.0.0.1:8080/api/tasks/YOUR_TASK_ID \
  -H "Content-Type: application/json" \
  -d '{"title": "Learn Vapor", "is_completed": true}'
```

Delete a task:

```bash
curl -X DELETE http://127.0.0.1:8080/api/tasks/YOUR_TASK_ID
```

## Adding Validation

Vapor makes input validation straightforward. Add a `CreateTaskRequest` DTO:

```swift
import Vapor

struct CreateTaskRequest: Content, Validatable {
    var title: String

    static func validations(_ validations: inout Validations) {
        validations.add("title", as: String.self, is: !.empty && .count(1...200))
    }
}
```

Then validate in your controller:

```swift
@Sendable
func create(req: Request) async throws -> TaskItem {
    try CreateTaskRequest.validate(content: req)
    let input = try req.content.decode(CreateTaskRequest.self)
    let task = TaskItem(title: input.title)
    try await task.save(on: req.db)
    return task
}
```

## Deploying

For a quick deployment, you can use Docker:

```dockerfile
FROM swift:6.0-jammy as build
WORKDIR /app
COPY . .
RUN swift build -c release

FROM swift:6.0-jammy-slim
COPY --from=build /app/.build/release/App /app/App
ENTRYPOINT ["/app/App"]
EXPOSE 8080
CMD ["serve", "--env", "production", "--hostname", "0.0.0.0"]
```

Build and run:

```bash
docker build -t task-api .
docker run -p 8080:8080 task-api
```

## What's Next?

From here you can extend your API with:

- **Authentication** using JWT or session tokens
- **PostgreSQL** for a production database
- **Middleware** for logging, CORS, and rate limiting
- **WebSockets** for real-time features
- **Testing** with XCTest and Vapor's test utilities

Vapor's documentation at [docs.vapor.codes](https://docs.vapor.codes) is excellent and covers all of these topics in depth.

## Wrapping Up

You just built a complete REST API in Swift — models, migrations, controllers, validation, and deployment. The entire stack is type-safe, async, and written in the same language as your iOS app. That's the power of server-side Swift with Vapor.

If you want to dive deeper, join us at the next Swift Riyadh meetup where we'll be building APIs live together.

*— Abdullah Alhaider*
