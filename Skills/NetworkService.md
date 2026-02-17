# Network Service Pattern

Modern networking with async/await in Swift.

---

## When to Use

- Building REST API clients
- Fetching and posting data to remote servers
- Need type-safe network requests
- Want testable network layer

---

## Core Pattern

### 1. Define Custom Error Type

```swift
enum NetworkError: LocalizedError {
    case invalidURL
    case invalidResponse
    case httpError(statusCode: Int)
    case decodingError
    case unknown(Error)
    
    var errorDescription: String? {
        switch self {
        case .invalidURL:
            return "The URL is invalid"
        case .invalidResponse:
            return "Invalid response from server"
        case .httpError(let statusCode):
            return "HTTP error: \(statusCode)"
        case .decodingError:
            return "Failed to decode response"
        case .unknown(let error):
            return "Unknown error: \(error.localizedDescription)"
        }
    }
}
```

### 2. Create Protocol

```swift
protocol NetworkServiceProtocol {
    func fetch<T: Decodable>(from endpoint: String) async throws -> T
    func post<T: Decodable, U: Encodable>(to endpoint: String, body: U) async throws -> T
}
```

### 3. Implement Service

```swift
@MainActor
class NetworkService: NetworkServiceProtocol {
    private let baseURL: String
    private let session: URLSession
    private let decoder: JSONDecoder
    private let encoder: JSONEncoder
    
    init(
        baseURL: String,
        session: URLSession = .shared,
        decoder: JSONDecoder = JSONDecoder(),
        encoder: JSONEncoder = JSONEncoder()
    ) {
        self.baseURL = baseURL
        self.session = session
        self.decoder = decoder
        self.encoder = encoder
        
        // Configure decoder
        self.decoder.dateDecodingStrategy = .iso8601
        self.decoder.keyDecodingStrategy = .convertFromSnakeCase
        
        // Configure encoder
        self.encoder.dateEncodingStrategy = .iso8601
        self.encoder.keyEncodingStrategy = .convertToSnakeCase
    }
    
    func fetch<T: Decodable>(from endpoint: String) async throws -> T {
        guard let url = URL(string: baseURL + endpoint) else {
            throw NetworkError.invalidURL
        }
        
        let (data, response) = try await session.data(from: url)
        
        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }
        
        guard (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.httpError(statusCode: httpResponse.statusCode)
        }
        
        do {
            return try decoder.decode(T.self, from: data)
        } catch {
            throw NetworkError.decodingError
        }
    }
    
    func post<T: Decodable, U: Encodable>(to endpoint: String, body: U) async throws -> T {
        guard let url = URL(string: baseURL + endpoint) else {
            throw NetworkError.invalidURL
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try encoder.encode(body)
        
        let (data, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }
        
        guard (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.httpError(statusCode: httpResponse.statusCode)
        }
        
        return try decoder.decode(T.self, from: data)
    }
}
```

---

## Usage Example

```swift
// Define your model
struct User: Codable, Identifiable {
    let id: Int
    let name: String
    let email: String
}

// Create service
let service = NetworkService(baseURL: "https://api.example.com")

// Fetch data
Task {
    do {
        let users: [User] = try await service.fetch(from: "/users")
        print("Fetched \(users.count) users")
    } catch {
        print("Error: \(error.localizedDescription)")
    }
}

// Post data
struct CreateUserRequest: Encodable {
    let name: String
    let email: String
}

Task {
    do {
        let newUser: User = try await service.post(
            to: "/users",
            body: CreateUserRequest(name: "John", email: "john@example.com")
        )
        print("Created user: \(newUser.name)")
    } catch {
        print("Error: \(error.localizedDescription)")
    }
}
```

---

## Best Practices

1. **Use protocols** for testability and dependency injection
2. **Configure JSON coders** with appropriate strategies (snake_case, ISO8601 dates)
3. **Handle all error cases** with custom error types
4. **Use async/await** instead of completion handlers or Combine
5. **Inject URLSession** for testing (mock sessions)
6. **Never hardcode API keys** - use configuration files or environment variables

---

## Testing Pattern

```swift
class MockNetworkService: NetworkServiceProtocol {
    var shouldFail = false
    var mockResponse: Any?
    
    func fetch<T: Decodable>(from endpoint: String) async throws -> T {
        if shouldFail {
            throw NetworkError.httpError(statusCode: 500)
        }
        
        guard let response = mockResponse as? T else {
            throw NetworkError.decodingError
        }
        
        return response
    }
    
    func post<T: Decodable, U: Encodable>(to endpoint: String, body: U) async throws -> T {
        if shouldFail {
            throw NetworkError.httpError(statusCode: 500)
        }
        
        guard let response = mockResponse as? T else {
            throw NetworkError.decodingError
        }
        
        return response
    }
}
```

---

## Common Variations

### Adding Headers

```swift
func fetch<T: Decodable>(from endpoint: String, headers: [String: String] = [:]) async throws -> T {
    var request = URLRequest(url: url)
    headers.forEach { request.setValue($1, forHTTPHeaderField: $0) }
    // ...
}
```

### Adding Authentication

```swift
class AuthenticatedNetworkService: NetworkServiceProtocol {
    private let baseService: NetworkServiceProtocol
    private let authToken: String
    
    init(baseService: NetworkServiceProtocol, authToken: String) {
        self.baseService = baseService
        self.authToken = authToken
    }
    
    func fetch<T: Decodable>(from endpoint: String) async throws -> T {
        // Add auth header to request
    }
}
```

---

## Key Takeaways

- ✅ Protocol-based for testability
- ✅ Generic methods for type safety
- ✅ Async/await for modern concurrency
- ✅ Proper error handling
- ✅ Configurable JSON coding strategies
- ✅ Dependency injection friendly
