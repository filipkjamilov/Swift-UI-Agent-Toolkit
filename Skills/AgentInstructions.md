# Agent Instructions for Swift/SwiftUI Development

This file contains instructions and guidelines for AI coding agents (like Claude) to follow when working with Swift and SwiftUI projects that use this template system.

---

## Table of Contents

1. [How to Use This Template System](#how-to-use-this-template-system)
2. [Understanding the Skills](#understanding-the-skills)
3. [Code Generation Guidelines](#code-generation-guidelines)
4. [Pattern Recognition](#pattern-recognition)
5. [Common Tasks and Responses](#common-tasks-and-responses)

---

## How to Use This Template System

### For AI Agents

When a user asks you to implement features in a Swift/SwiftUI project that contains this template:

1. **Always read CLAUDE.md first** to understand the coding standards
2. **Reference the appropriate skill file** for the pattern being implemented
3. **Follow the established patterns** shown in the skills
4. **Maintain consistency** with the existing codebase style

### Project Structure Recognition

When you see this structure:
```
Project/
├── CLAUDE.md
├── Skills/
│   ├── README.md
│   ├── NetworkService.swift
│   ├── MVVMTemplate.swift
│   ├── StateManagement.swift
│   ├── TestingTemplate.swift
│   ├── CommonUIComponents.swift
│   ├── NavigationPatterns.swift
│   └── AgentInstructions.md
```

You should recognize this as a project using the Claude template system and automatically apply these best practices.

---

## Understanding the Skills

### Skill Selection Matrix

Use this matrix to determine which skill to reference:

| User Request | Primary Skill | Secondary Skills | Action |
|--------------|---------------|------------------|--------|
| "Add a screen that shows..." | MVVMTemplate.swift | NavigationPatterns.swift, CommonUIComponents.swift | Create Model, ViewModel, View following MVVM pattern |
| "Fetch data from API..." | NetworkService.swift | MVVMTemplate.swift, TestingTemplate.swift | Implement service protocol, create ViewModel to consume it |
| "Add navigation to..." | NavigationPatterns.swift | MVVMTemplate.swift | Use NavigationCoordinator pattern |
| "Show a loading state..." | CommonUIComponents.swift | StateManagement.swift | Use LoadingView, manage state with @Published |
| "Add a form with..." | StateManagement.swift | CommonUIComponents.swift | Use @State for form fields, CustomTextField component |
| "Write tests for..." | TestingTemplate.swift | - | Use Swift Testing framework with #expect |
| "Pass data between views..." | StateManagement.swift | NavigationPatterns.swift | Use appropriate property wrapper (@Binding, @EnvironmentObject, etc.) |
| "Add tabs to the app..." | NavigationPatterns.swift | StateManagement.swift | Implement TabView with NavigationStack per tab |

### Reading the Skills

Each skill file contains:
- **MARK comments** separating different sections
- **Inline documentation** explaining patterns
- **Working examples** you can adapt
- **Usage comments** showing how to implement
- **Best practices sections** with do's and don'ts

---

## Code Generation Guidelines

### 1. MVVM Implementation

When implementing a new feature screen:

```swift
// Step 1: Read MVVMTemplate.swift

// Step 2: Identify the data model
struct YourModel: Identifiable, Codable {
    let id: UUID
    // ... properties based on requirements
}

// Step 3: Create ViewModel following the pattern
@MainActor
class YourViewModel: ObservableObject {
    @Published var items: [YourModel] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private let service: YourServiceProtocol
    
    init(service: YourServiceProtocol) {
        self.service = service
    }
    
    func loadData() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            items = try await service.fetchItems()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}

// Step 4: Create View using common components
struct YourView: View {
    @StateObject private var viewModel = YourViewModel()
    
    var body: some View {
        Group {
            if viewModel.isLoading {
                LoadingView() // From CommonUIComponents
            } else if viewModel.items.isEmpty {
                EmptyStateView(title: "No items")
            } else {
                List(viewModel.items) { item in
                    // Item row
                }
            }
        }
        .task {
            await viewModel.loadData()
        }
    }
}
```

### 2. Network Requests

When implementing API calls:

```swift
// Step 1: Reference NetworkService.swift

// Step 2: Define the model matching the API response
struct APIResponse: Codable {
    let data: [YourModel]
}

// Step 3: Create protocol for the service
protocol YourServiceProtocol {
    func fetchItems() async throws -> [YourModel]
}

// Step 4: Implement using NetworkService pattern
class YourService: YourServiceProtocol {
    private let networkService: NetworkServiceProtocol
    
    init(networkService: NetworkServiceProtocol = NetworkService(baseURL: "https://api.example.com")) {
        self.networkService = networkService
    }
    
    func fetchItems() async throws -> [YourModel] {
        let response: APIResponse = try await networkService.fetch(from: "/items")
        return response.data
    }
}
```

### 3. Navigation Implementation

When adding navigation:

```swift
// Step 1: Reference NavigationPatterns.swift

// Step 2: Define routes in existing AppRoute enum or create new one
enum AppRoute: Hashable {
    case yourNewScreen(id: String)
    case anotherScreen
}

// Step 3: Use NavigationCoordinator pattern
@MainActor
class NavigationCoordinator: ObservableObject {
    @Published var path = NavigationPath()
    
    func navigate(to route: AppRoute) {
        path.append(route)
    }
    
    func navigateBack() {
        path.removeLast()
    }
    
    func navigateToRoot() {
        path.removeLast(path.count)
    }
}

// Step 4: Set up NavigationStack with destinations
NavigationStack(path: $coordinator.path) {
    YourRootView()
        .navigationDestination(for: AppRoute.self) { route in
            switch route {
            case .yourNewScreen(let id):
                YourScreen(id: id)
            case .anotherScreen:
                AnotherScreen()
            }
        }
}
```

### 4. State Management

When choosing property wrappers:

```swift
// Reference StateManagement.swift for detailed examples

// View-local state (simple values)
@State private var isExpanded = false
@State private var text = ""

// Two-way binding to parent
@Binding var isPresented: Bool

// Creating and owning an ObservableObject
@StateObject private var viewModel = YourViewModel()

// Observing an ObservableObject from parent
@ObservedObject var viewModel: YourViewModel

// Shared object across view hierarchy
@EnvironmentObject private var appState: AppState

// System environment values
@Environment(\.dismiss) private var dismiss
@Environment(\.colorScheme) private var colorScheme
```

### 5. UI Components

When building UI:

```swift
// Step 1: Reference CommonUIComponents.swift

// Step 2: Use existing components when possible
LoadingView(message: "Loading data...")
EmptyStateView(title: "No results", subtitle: "Try a different search")
ErrorView(error: error) { /* retry action */ }

// Step 3: Compose with CardView for consistent styling
CardView {
    VStack(alignment: .leading) {
        Text("Title")
            .font(.headline)
        Text("Content")
            .foregroundStyle(.secondary)
    }
}

// Step 4: Use custom button styles
Button("Primary Action") { }
    .buttonStyle(PrimaryButtonStyle())

Button("Secondary Action") { }
    .buttonStyle(SecondaryButtonStyle())
```

### 6. Testing

When writing tests:

```swift
// Reference TestingTemplate.swift

import Testing
@testable import YourApp

struct YourViewModelTests {
    // Basic test
    @Test("ViewModel loads data successfully")
    func testLoadData() async throws {
        let mockService = MockYourService()
        let viewModel = YourViewModel(service: mockService)
        
        await viewModel.loadData()
        
        #expect(viewModel.items.count > 0)
        #expect(viewModel.isLoading == false)
        #expect(viewModel.errorMessage == nil)
    }
    
    // Error handling test
    @Test("ViewModel handles errors correctly")
    func testErrorHandling() async throws {
        let mockService = MockYourService(shouldFail: true)
        let viewModel = YourViewModel(service: mockService)
        
        await viewModel.loadData()
        
        #expect(viewModel.items.isEmpty)
        #expect(viewModel.errorMessage != nil)
    }
}

// Mock service for testing
class MockYourService: YourServiceProtocol {
    var shouldFail = false
    var mockData: [YourModel] = []
    
    func fetchItems() async throws -> [YourModel] {
        if shouldFail {
            throw NSError(domain: "TestError", code: 500)
        }
        return mockData
    }
}
```

---

## Pattern Recognition

### When You See...

#### User asks: "Add a list of items from an API"

**Pattern to apply:**
1. Create Model (Codable, Identifiable)
2. Create Service Protocol
3. Implement Service using NetworkService pattern
4. Create ViewModel using MVVM pattern
5. Create View with List and loading states
6. Write tests for ViewModel and Service

**Files to reference:**
- NetworkService.swift
- MVVMTemplate.swift
- CommonUIComponents.swift
- TestingTemplate.swift

#### User asks: "Navigate to detail screen when tapping item"

**Pattern to apply:**
1. Define route in AppRoute enum
2. Add NavigationCoordinator if not present
3. Set up NavigationStack with navigationDestination
4. Create detail view
5. Add navigation action on tap

**Files to reference:**
- NavigationPatterns.swift
- MVVMTemplate.swift

#### User asks: "Add a form to create/edit something"

**Pattern to apply:**
1. Use @State for form fields
2. Use CustomTextField components
3. Present as sheet
4. Add validation
5. Submit through ViewModel

**Files to reference:**
- StateManagement.swift
- CommonUIComponents.swift
- MVVMTemplate.swift

#### User asks: "Share data between screens"

**Pattern to apply:**
1. For parent-child: Use @Binding
2. For hierarchical: Pass through NavigationStack
3. For app-wide: Use @EnvironmentObject
4. For simple values: Pass as parameters

**Files to reference:**
- StateManagement.swift
- NavigationPatterns.swift

---

## Common Tasks and Responses

### Task: "Implement user authentication"

**Agent Response:**
```
I'll implement authentication following the MVVM pattern from MVVMTemplate.swift:

1. Create User model and AuthState
2. Create AuthService protocol with login/logout methods
3. Implement AuthService using NetworkService pattern
4. Create AuthViewModel managing authentication state
5. Create login/signup views
6. Use @EnvironmentObject for app-wide auth state
7. Add navigation guards based on auth state

Let me start by creating the models and service...
```

### Task: "Add pull-to-refresh"

**Agent Response:**
```
I'll add pull-to-refresh using the pattern from MVVMTemplate.swift:

The template shows using .refreshable modifier with async methods.
I'll add this to your view:

.refreshable {
    await viewModel.refreshData()
}

And add the refresh method to your ViewModel...
```

### Task: "Show error messages"

**Agent Response:**
```
I'll implement error handling using patterns from MVVMTemplate.swift and CommonUIComponents.swift:

1. Add @Published var errorMessage: String? to ViewModel
2. Use ErrorView component for full-screen errors
3. Or use .alert modifier for simple error alerts
4. Include retry mechanism

Here's the implementation...
```

### Task: "Add search functionality"

**Agent Response:**
```
Following the search pattern in MVVMTemplate.swift:

1. Add @Published var searchQuery = "" to ViewModel
2. Create computed property for filtered results
3. Use .searchable modifier on view
4. Optionally debounce search for API calls

Implementation:
```

---

## Code Style Enforcement

### Always Apply These Rules

1. **Naming Conventions** (from CLAUDE.md)
   - Types: PascalCase
   - Properties/Methods: camelCase
   - Booleans: is/has/should prefix

2. **Property Wrappers** (from StateManagement.swift)
   - Use `private` when possible
   - Choose correct wrapper for use case
   - Never force unwrap

3. **Async/Await** (from CLAUDE.md)
   - Prefer async/await over Combine
   - Use `defer` for cleanup
   - Handle errors with do-catch

4. **Architecture** (from MVVMTemplate.swift)
   - Separate Model, View, ViewModel
   - Protocol-based services
   - Dependency injection for testability

5. **UI Components** (from CommonUIComponents.swift)
   - Use existing components when available
   - Follow established styling patterns
   - Include accessibility labels

6. **Testing** (from TestingTemplate.swift)
   - Use Swift Testing framework
   - Test ViewModels with mock services
   - Use #expect instead of XCTAssert

---

## Error Prevention

### Common Mistakes to Avoid

❌ **DON'T:**
```swift
// Nesting NavigationStacks
NavigationStack {
    NavigationStack { // ❌ Never nest
        // ...
    }
}

// Force unwrapping
let user = users.first! // ❌ Dangerous

// Using Combine when async/await works
URLSession.shared.dataTaskPublisher(for: url) // ❌ Outdated pattern

// Hardcoding API keys
let apiKey = "sk_live_123456" // ❌ Security risk

// Ignoring accessibility
Button(action: { }) {
    Image(systemName: "trash") // ❌ No label
}
```

✅ **DO:**
```swift
// One NavigationStack per tab
TabView {
    NavigationStack { /* Tab 1 */ }
    NavigationStack { /* Tab 2 */ }
}

// Safe optional handling
guard let user = users.first else { return }

// Modern async/await
let (data, _) = try await URLSession.shared.data(from: url)

// Secure configuration
let apiKey = Bundle.main.object(forInfoDictionaryKey: "API_KEY") as? String

// Proper accessibility
Button(action: { }) {
    Image(systemName: "trash")
}
.accessibilityLabel("Delete item")
```

---

## Response Templates

### When user asks to implement a feature

```
I'll implement [feature] following the [pattern name] from [SkillFile.swift].

Here's my approach:
1. [Step 1 with reference to skill]
2. [Step 2 with reference to skill]
3. [Step 3 with reference to skill]

Let me start by [first concrete action]...
```

### When user asks about best practices

```
According to CLAUDE.md and [relevant skill], the best practice for [topic] is:

[Explanation with code example from skill]

This approach ensures:
- [Benefit 1]
- [Benefit 2]
- [Benefit 3]
```

### When user asks to fix a bug

```
I see the issue. Looking at [SkillFile.swift], the correct pattern is:

[Show correct pattern]

The problem in your code is [explanation]. Let me fix this...
```

---

## Integration Checklist

Before generating any code, verify:

- [ ] I've read CLAUDE.md for code style
- [ ] I've identified the relevant skill file(s)
- [ ] I'm using the correct property wrapper for state
- [ ] I'm following MVVM architecture
- [ ] I'm using async/await, not Combine
- [ ] I'm including error handling
- [ ] I'm using existing UI components when possible
- [ ] I'm including accessibility modifiers
- [ ] The code is testable with dependency injection
- [ ] I'm following Swift naming conventions

---

## Advanced Patterns

### Combining Multiple Skills

When implementing complex features, combine skills:

**Example: "Add a screen that fetches and displays user profiles with navigation"**

```swift
// 1. Model (MVVMTemplate.swift)
struct UserProfile: Identifiable, Codable {
    let id: String
    let name: String
    let email: String
}

// 2. Service (NetworkService.swift)
protocol UserProfileServiceProtocol {
    func fetchProfiles() async throws -> [UserProfile]
}

class UserProfileService: UserProfileServiceProtocol {
    private let networkService: NetworkServiceProtocol
    
    init(networkService: NetworkServiceProtocol = NetworkService(baseURL: "https://api.example.com")) {
        self.networkService = networkService
    }
    
    func fetchProfiles() async throws -> [UserProfile] {
        try await networkService.fetch(from: "/profiles")
    }
}

// 3. ViewModel (MVVMTemplate.swift + StateManagement.swift)
@MainActor
class UserProfileListViewModel: ObservableObject {
    @Published var profiles: [UserProfile] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private let service: UserProfileServiceProtocol
    
    init(service: UserProfileServiceProtocol = UserProfileService()) {
        self.service = service
    }
    
    func loadProfiles() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            profiles = try await service.fetchProfiles()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}

// 4. Navigation (NavigationPatterns.swift)
enum AppRoute: Hashable {
    case profileDetail(profile: UserProfile)
}

@MainActor
class NavigationCoordinator: ObservableObject {
    @Published var path = NavigationPath()
    
    func navigate(to route: AppRoute) {
        path.append(route)
    }
}

// 5. View (MVVMTemplate.swift + CommonUIComponents.swift + NavigationPatterns.swift)
struct UserProfileListView: View {
    @StateObject private var viewModel = UserProfileListViewModel()
    @EnvironmentObject private var coordinator: NavigationCoordinator
    
    var body: some View {
        Group {
            if viewModel.isLoading {
                LoadingView() // CommonUIComponents
            } else if viewModel.profiles.isEmpty {
                EmptyStateView(title: "No profiles") // CommonUIComponents
            } else {
                List(viewModel.profiles) { profile in
                    Button {
                        coordinator.navigate(to: .profileDetail(profile: profile))
                    } label: {
                        CardView { // CommonUIComponents
                            HStack {
                                AvatarView(initials: String(profile.name.prefix(2))) // CommonUIComponents
                                VStack(alignment: .leading) {
                                    Text(profile.name)
                                        .font(.headline)
                                    Text(profile.email)
                                        .font(.subheadline)
                                        .foregroundStyle(.secondary)
                                }
                            }
                        }
                    }
                }
            }
        }
        .navigationTitle("Profiles")
        .task {
            await viewModel.loadProfiles()
        }
        .refreshable {
            await viewModel.loadProfiles()
        }
    }
}

// 6. Tests (TestingTemplate.swift)
import Testing
@testable import YourApp

struct UserProfileListViewModelTests {
    @Test("Successfully loads profiles")
    func testLoadProfiles() async throws {
        let mockService = MockUserProfileService()
        mockService.mockProfiles = [
            UserProfile(id: "1", name: "John", email: "john@example.com")
        ]
        
        let viewModel = UserProfileListViewModel(service: mockService)
        await viewModel.loadProfiles()
        
        #expect(viewModel.profiles.count == 1)
        #expect(viewModel.isLoading == false)
        #expect(viewModel.errorMessage == nil)
    }
}
```

---

## Updating Agent Behavior

As SwiftUI evolves, agents should:

1. **Check Apple Documentation** using DocumentationSearch
2. **Update skill files** when new APIs are available
3. **Deprecate old patterns** and note them in CLAUDE.md
4. **Add new skills** for new architectural patterns
5. **Maintain backward compatibility** notes in skills

---

## Summary for AI Agents

**When working with a project containing this template:**

1. ✅ Always reference CLAUDE.md for standards
2. ✅ Use the skills as your implementation guide
3. ✅ Follow established patterns consistently
4. ✅ Combine skills for complex features
5. ✅ Write testable, accessible code
6. ✅ Use modern Swift concurrency (async/await)
7. ✅ Apply MVVM architecture
8. ✅ Include proper error handling
9. ✅ Use type-safe navigation
10. ✅ Reuse common UI components

**The goal:** Generate code that looks like it was written by a developer who has been following these patterns from day one.

---

**Last Updated**: February 2026
