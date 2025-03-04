# CLAUDE.md - MCP Java SDK Development Guide

## Project Overview
The MCP Java SDK implements the [Model Context Protocol](https://modelcontextprotocol.org/docs/concepts/architecture) for Java applications. This SDK enables communication between Java applications and AI models/tools through a standardized interface, supporting both synchronous and asynchronous patterns.

Key components:
- Core MCP client and server implementations
- Transport mechanisms (stdio, HTTP/SSE)
- Spring integration (WebFlux and WebMVC)
- Reactive support using Project Reactor

## Build/Test/Lint Commands
```bash
# Build project without running tests
./mvnw clean install -DskipTests

# Run all tests (requires Docker and npx)
./mvnw test

# Run a single test class
./mvnw test -Dtest=McpClientProtocolVersionTests

# Run a specific test method
./mvnw test -Dtest=AbstractMcpAsyncClientTests#testPing

# Format code using Spring Java Format
./mvnw spring-javaformat:format

# Validate code formatting
./mvnw spring-javaformat:validate
```

## Quick Start Guide

### Creating a Synchronous MCP Client
```java
// Create a transport instance (e.g., for Server-Sent Events)
ClientMcpTransport transport = new HttpClientSseClientTransport("http://localhost:8080/mcp");

// Build a synchronous client
McpSyncClient client = McpClient.sync(transport)
    .requestTimeout(Duration.ofSeconds(5))
    .build();

// Initialize the client
client.initialize();

// Call a tool
Map<String, Object> arguments = Map.of("prompt", "What is the capital of France?");
CallToolResult result = client.callTool("ai-response", arguments);

// Clean up
client.close();
```

### Creating an Asynchronous MCP Client
```java
// Create a transport instance
ClientMcpTransport transport = new HttpClientSseClientTransport("http://localhost:8080/mcp");

// Build an asynchronous client with callbacks
McpAsyncClient client = McpClient.async(transport)
    .requestTimeout(Duration.ofSeconds(10))
    .toolsChangeConsumer(tools -> Mono.fromRunnable(() -> 
        System.out.println("Tools updated: " + tools)))
    .build();

// Initialize the client (returns CompletableFuture)
client.initialize().thenAccept(result -> {
    System.out.println("Connected to: " + result.serverInfo().name());
});

// Call a tool asynchronously
Map<String, Object> arguments = Map.of("query", "weather");
client.callTool("search", arguments).thenAccept(result -> {
    System.out.println("Tool result: " + result.content());
});

// Clean up when done
client.close();
```

## Code Style Guidelines

### Java
- Java Version: 17 (as specified in pom.xml)
- Class names: PascalCase (e.g., `McpClient`)
- Method names: camelCase (e.g., `addRoot()`)
- Constants: UPPER_SNAKE_CASE (e.g., `TIMEOUT`)
- Interfaces preferred over abstract classes when possible
- Comprehensive Javadoc for all public classes and methods
- Builder pattern for complex object construction
- Use `Assert` utility for null checks and preconditions
- Prefer immutable objects when possible
- Use reactive paradigms with Reactor for async operations
- Import order: java.*, javax.*, org.*, com.*
- Always handle exceptions properly using try-catch or throws
- Line length: 100 characters maximum