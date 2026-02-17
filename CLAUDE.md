# Swift & SwiftUI Best Practices Guide

This guide serves as a comprehensive reference for building modern iOS, iPadOS, macOS, watchOS, and visionOS applications using Swift and SwiftUI.

## Table of Contents

1. [Code Style Guidelines](#code-style-guidelines)
2. [SwiftUI Architecture Patterns](#swiftui-architecture-patterns)
3. [State Management](#state-management)
4. [Async/Await Patterns](#asyncawait-patterns)
5. [Testing Strategy](#testing-strategy)
6. [Performance Best Practices](#performance-best-practices)
7. [Accessibility](#accessibility)
8. [Security Considerations](#security-considerations)

---

## Code Style Guidelines

### Naming Conventions

- **Types**: PascalCase (structs, classes, enums, protocols)
  ```swift
  struct UserProfile { }
  class NetworkManager { }
  enum AppState { }
  protocol Refreshable { }
  ```

- **Properties & Methods**: camelCase
  ```swift
  var userName: String
  func fetchUserData() async throws -> User
  ```

- **Constants**: camelCase for properties, UPPER_SNAKE_CASE for static/global constants
  ```swift
  let maxRetryCount = 3
  static let API_BASE_URL = "https://api.example.com"
  ```

- **Booleans**: Use is/has/should prefixes
  ```swift
  var isLoading: Bool
  var hasCompletedOnboarding: Bool
  var shouldShowAlert: Bool
  ```

### Formatting

- **Indentation**: 4 spaces (never tabs)
- **Line Length**: Aim for 100-120 characters maximum
- **Braces**: Opening brace on same line, closing brace on new line
- **Spacing**: One blank line between methods, two between types

### Imports

- Keep imports minimal and organized
- Import only what you need (avoid `import UIKit` when `import SwiftUI` suffices)
- Group imports: system frameworks first, then third-party

```swift
import SwiftUI
import Combine

import Alamofire
```

---

## SwiftUI Architecture Patterns

### MVVM (Model-View-ViewModel)

SwiftUI naturally aligns with MVVM architecture:

```swift
// Model
struct User: Identifiable, Codable {
    let id: UUID
    let name: String
    let email: String
}

// ViewModel
@MainActor
class UserViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private let service: UserService
    
    init(service: UserService = UserService()) {
        self.service = service
    }
    
    func loadUsers() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            users = try await service.fetchUsers()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}

// View
struct UserListView: View {
    @StateObject private var viewModel = UserViewModel()
    
    var body: some View {
        List(viewModel.users) { user in
            UserRow(user: user)
        }
        .overlay {
            if viewModel.isLoading {
                ProgressView()
            }
        }
        .task {
            await viewModel.loadUsers()
        }
    }
}
```

### Separation of Concerns

- **Views**: UI presentation only
- **ViewModels**: Business logic and state management
- **Models**: Data structures
- **Services**: Network, persistence, external integrations

---

## State Management

### Property Wrappers

Choose the right property wrapper for your needs:

#### @State
- **Use for**: Simple, view-local state
- **Ownership**: View owns the data
```swift
@State private var isExpanded = false
@State private var username = ""
```

#### @StateObject
- **Use for**: Creating and owning ObservableObject instances
- **Ownership**: View owns the object
```swift
@StateObject private var viewModel = ContentViewModel()
```

#### @ObservedObject
- **Use for**: Observing ObservableObject passed from parent
- **Ownership**: Parent view owns the object
```swift
@ObservedObject var viewModel: ContentViewModel
```

#### @EnvironmentObject
- **Use for**: Sharing objects across view hierarchy
- **Ownership**: Ancestor view provides the object
```swift
@EnvironmentObject private var appState: AppState
```

#### @Binding
- **Use for**: Two-way connection to parent's state
- **Ownership**: Parent view owns the data
```swift
@Binding var isPresented: Bool
```

#### @Environment
- **Use for**: Reading system or custom environment values
```swift
@Environment(\.dismiss) private var dismiss
@Environment(\.colorScheme) private var colorScheme
```

### State Best Practices

1. **Keep state minimal**: Only store what you need
2. **Use private when possible**: Encapsulate internal state
3. **Prefer value types**: Use structs over classes when possible
4. **Avoid force unwrapping**: Use optionals safely with if-let or guard-let

---

## Async/Await Patterns

### Prefer Swift Concurrency over Combine

Modern Swift favors async/await over Combine for asynchronous operations:

```swift
// Good: Using async/await
func fetchData() async throws -> Data {
    let (data, _) = try await URLSession.shared.data(from: url)
    return data
}

// Avoid: Using Combine unless specifically needed
func fetchDataPublisher() -> AnyPublisher<Data, Error> {
    URLSession.shared.dataTaskPublisher(for: url)
        .map(\.data)
        .eraseToAnyPublisher()
}
```

### Task Management

```swift
struct ContentView: View {
    @State private var data: [Item] = []
    
    var body: some View {
        List(data) { item in
            Text(item.name)
        }
        .task {
            // Automatically cancelled when view disappears
            await loadData()
        }
    }
    
    func loadData() async {
        do {
            data = try await apiClient.fetchItems()
        } catch {
            print("Error loading data: \(error)")
        }
    }
}
```

### Structured Concurrency

```swift
// Run tasks concurrently
func loadMultipleResources() async throws -> (users: [User], posts: [Post]) {
    async let users = userService.fetchUsers()
    async let posts = postService.fetchPosts()
    
    return try await (users, posts)
}

// Task groups for dynamic concurrency
func processItems(_ items: [Item]) async throws -> [ProcessedItem] {
    try await withThrowingTaskGroup(of: ProcessedItem.self) { group in
        for item in items {
            group.addTask {
                try await process(item)
            }
        }
        
        var results: [ProcessedItem] = []
        for try await result in group {
            results.append(result)
        }
        return results
    }
}
```

---

## Testing Strategy

### Use Swift Testing Framework

Prefer the modern Swift Testing framework over XCTest:

```swift
import Testing
@testable import Claude

struct UserViewModelTests {
    @Test func loadUsersSuccessfully() async throws {
        let mockService = MockUserService()
        let viewModel = UserViewModel(service: mockService)
        
        await viewModel.loadUsers()
        
        #expect(viewModel.users.count == 2)
        #expect(viewModel.isLoading == false)
    }
    
    @Test func loadUsersWithError() async throws {
        let mockService = MockUserService(shouldFail: true)
        let viewModel = UserViewModel(service: mockService)
        
        await viewModel.loadUsers()
        
        #expect(viewModel.errorMessage != nil)
        #expect(viewModel.users.isEmpty)
    }
}
```

### UI Testing with XCUITest

```swift
import XCTest

final class ClaudeUITests: XCTestCase {
    let app = XCUIApplication()
    
    override func setUpWithError() throws {
        continueAfterFailure = false
        app.launch()
    }
    
    func testUserCanNavigateToProfile() throws {
        let profileButton = app.buttons["Profile"]
        XCTAssertTrue(profileButton.exists)
        
        profileButton.tap()
        
        let profileTitle = app.staticTexts["My Profile"]
        XCTAssertTrue(profileTitle.waitForExistence(timeout: 2))
    }
}
```

---

## Performance Best Practices

### View Performance

1. **Avoid unnecessary body evaluations**
```swift
// Good: Extract to computed property
private var filteredItems: [Item] {
    items.filter { $0.isActive }
}

// Avoid: Computing in body repeatedly
var body: some View {
    List(items.filter { $0.isActive }) { item in
        // ...
    }
}
```

2. **Use lazy stacks for long lists**
```swift
LazyVStack {
    ForEach(items) { item in
        ItemRow(item: item)
    }
}
```

3. **Identify views with explicit IDs**
```swift
ForEach(items, id: \.id) { item in
    ItemRow(item: item)
}
```

### Image Loading

```swift
// Use AsyncImage for remote images
AsyncImage(url: URL(string: imageURL)) { phase in
    switch phase {
    case .empty:
        ProgressView()
    case .success(let image):
        image
            .resizable()
            .scaledToFit()
    case .failure:
        Image(systemName: "photo")
    @unknown default:
        EmptyView()
    }
}
```

### Memory Management

- Use `weak` or `unowned` references to avoid retain cycles
- Be mindful of capturing `self` in closures
```swift
Task { [weak self] in
    guard let self else { return }
    await self.performAction()
}
```

---

## Accessibility

### Always Consider Accessibility

```swift
Button(action: deleteItem) {
    Image(systemName: "trash")
}
.accessibilityLabel("Delete item")
.accessibilityHint("Removes this item from your list")

Text("Loading...")
    .accessibilityAddTraits(.updatesFrequently)

Toggle("Dark Mode", isOn: $isDarkMode)
    .accessibilityValue(isDarkMode ? "On" : "Off")
```

### Dynamic Type Support

```swift
Text("Welcome")
    .font(.headline)
    .dynamicTypeSize(...DynamicTypeSize.xxxLarge)
```

---

## Security Considerations

### Never Hardcode Secrets

```swift
// Bad
let apiKey = "sk_live_123456789"

// Good: Use Configuration or environment
let apiKey = Bundle.main.object(forInfoDictionaryKey: "API_KEY") as? String
```

### Validate User Input

```swift
func validateEmail(_ email: String) -> Bool {
    let emailRegex = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}"
    let predicate = NSPredicate(format: "SELF MATCHES %@", emailRegex)
    return predicate.evaluate(with: email)
}
```

### Use Keychain for Sensitive Data

```swift
// Store sensitive data in Keychain, not UserDefaults
import Security

func saveToKeychain(key: String, data: Data) throws {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key,
        kSecValueData as String: data
    ]
    
    SecItemDelete(query as CFDictionary)
    let status = SecItemAdd(query as CFDictionary, nil)
    
    guard status == errSecSuccess else {
        throw KeychainError.saveFailed
    }
}
```

---

## Additional Resources

- **Apple Developer Documentation**: Use `DocumentationSearch` in Xcode
- **Swift Evolution**: https://github.com/apple/swift-evolution
- **WWDC Sessions**: https://developer.apple.com/videos/

## Skills Directory

See the `Skills/` directory for practical templates implementing these patterns:
- **NetworkService.swift**: Network requests with async/await
- **MVVMTemplate.swift**: MVVM architecture
- **StateManagement.swift**: State management patterns
- **CommonUIComponents.swift**: Common UI components
- **NavigationPatterns.swift**: Complex navigation patterns
- **TestingTemplate.swift**: Testing examples
- **AgentInstructions.md**: Guidelines for AI agents working with this template

### For AI Coding Agents

If you're an AI agent (like Claude) working on this project, read `Skills/AgentInstructions.md` first. It contains:
- Which skill to use for each type of request
- Code generation patterns to follow
- Integration examples combining multiple skills
- Common mistakes to avoid
- Response templates for consistency

---

**Last Updated**: February 2026
**Maintained By**: Your Team
