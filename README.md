# SwiftUI Agent Toolkit

> Best practices, patterns, and architectural guides for building modern Swift/SwiftUI applications with AI-assisted development.

[![Swift](https://img.shields.io/badge/Swift-5.9+-orange.svg)](https://swift.org)
[![SwiftUI](https://img.shields.io/badge/SwiftUI-iOS%2017+-blue.svg)](https://developer.apple.com/xcode/swiftui/)

---

## 📖 Overview

This toolkit provides comprehensive guides for building production-ready Swift/SwiftUI applications. Each skill is a focused markdown document with patterns, examples, and best practices designed to help both developers and AI coding agents (like Claude, Cursor, GitHub Copilot) generate consistent, high-quality code.

## 🎯 What's Inside

### Core Documentation

- **[CLAUDE.md](CLAUDE.md)** - Complete Swift/SwiftUI coding standards and best practices
- **[Skills/](Skills/)** - Individual skill guides for specific patterns

### Skills

| Skill | Description | Key Topics |
|-------|-------------|------------|
| **[NetworkService](Skills/NetworkService.md)** | Modern networking with async/await | REST APIs, error handling, JSON decoding, mock services |
| **[MVVMPattern](Skills/MVVMPattern.md)** | Complete MVVM architecture | Models, ViewModels, Views, dependency injection, testing |
| **[StateManagement](Skills/StateManagement.md)** | SwiftUI property wrappers guide | @State, @Binding, @StateObject, @EnvironmentObject, decision trees |
| **[Navigation](Skills/Navigation.md)** | Complex navigation patterns | NavigationStack, sheets, deep linking, coordinators |
| **[Testing](Skills/Testing.md)** | Swift Testing framework | Unit tests, async testing, mocks, parameterized tests |
| **[UIComponents](Skills/UIComponents.md)** | Reusable UI component patterns | Loading states, empty states, custom buttons, cards |
| **[AgentInstructions](Skills/AgentInstructions.md)** | Guidelines for AI coding agents | Skill selection, code generation, pattern recognition |

---

## 🚀 Quick Start

### For Developers

1. **Clone or download this repository**
   ```bash
   git clone https://github.com/filipkjamilov/Swift-UI-Agent-Toolkit.git
   ```

2. **Copy to your project**
   ```bash
   cp -r Swift-UI-Agent-Toolkit/CLAUDE.md your-project/
   cp -r Swift-UI-Agent-Toolkit/Skills your-project/
   ```

3. **Reference when building**
   - Check the relevant skill before implementing a feature
   - Copy code examples as starting points
   - Follow established patterns for consistency

### For AI Coding Agents

If you're an AI agent (Claude, Cursor, etc.) working on a Swift/SwiftUI project:

1. **Read [AgentInstructions.md](Skills/AgentInstructions.md)** first
2. **Use the skill selection matrix** to choose appropriate patterns
3. **Follow code generation guidelines** for consistency
4. **Combine skills** as shown in integration examples

---

## 💡 Usage Examples

### Building a Feature

Most features combine multiple skills:

```swift
// 1. Model (MVVMPattern.md)
struct User: Identifiable, Codable {
    let id: UUID
    let name: String
    let email: String
}

// 2. Service (NetworkService.md)
protocol UserServiceProtocol {
    func fetchUsers() async throws -> [User]
}

class UserService: UserServiceProtocol {
    private let networkService: NetworkServiceProtocol
    
    func fetchUsers() async throws -> [User] {
        try await networkService.fetch(from: "/users")
    }
}

// 3. ViewModel (MVVMPattern.md + StateManagement.md)
@MainActor
class UserListViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private let service: UserServiceProtocol
    
    init(service: UserServiceProtocol) {
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

// 4. View (UIComponents.md + Navigation.md)
struct UserListView: View {
    @StateObject private var viewModel = UserListViewModel(service: UserService())
    
    var body: some View {
        Group {
            if viewModel.isLoading {
                LoadingView() // From UIComponents.md
            } else if let error = viewModel.errorMessage {
                ErrorView(error: NSError(domain: "", code: 0)) {
                    Task { await viewModel.loadUsers() }
                }
            } else {
                List(viewModel.users) { user in
                    UserRow(user: user)
                }
            }
        }
        .task {
            await viewModel.loadUsers()
        }
    }
}

// 5. Tests (Testing.md)
struct UserListViewModelTests {
    @Test("Loads users successfully")
    @MainActor
    func testLoadUsers() async throws {
        let mockService = MockUserService()
        let viewModel = UserListViewModel(service: mockService)
        
        await viewModel.loadUsers()
        
        #expect(viewModel.users.count > 0)
        #expect(viewModel.isLoading == false)
    }
}
```

---

## 🏗️ Architecture Principles

All skills follow these core principles:

✅ **Modern Swift Concurrency** - Use async/await instead of completion handlers  
✅ **Protocol-Oriented** - Protocols for testability and flexibility  
✅ **Type Safety** - Leverage Swift's type system  
✅ **Error Handling** - Proper do-catch and custom error types  
✅ **Separation of Concerns** - Clear boundaries between layers  
✅ **Accessibility** - Support VoiceOver and Dynamic Type  
✅ **Testability** - Dependency injection and mock services  
✅ **Documentation** - Clear examples and best practices  

---

## 🎓 Who Is This For?

### Developers
- Learning SwiftUI best practices
- Building production iOS/macOS apps
- Want consistent, maintainable code
- Need architecture guidance

### Teams
- Establishing coding standards
- Onboarding new developers
- Creating internal documentation
- Ensuring code consistency

### AI Agents
- Generating SwiftUI code
- Following architectural patterns
- Maintaining code quality
- Understanding project context

---

## 📚 What Each Skill Covers

### NetworkService.md
- Protocol-based architecture
- GET/POST requests with generics
- Custom error handling
- JSON encoding/decoding
- Mock services for testing

### MVVMPattern.md
- Model-View-ViewModel architecture
- Service layer protocols
- Loading and error states
- Search and filtering
- Complete CRUD examples

### StateManagement.md
- Property wrapper decision tree
- @State, @Binding, @StateObject
- @ObservedObject, @EnvironmentObject
- @Environment and custom values
- Best practices and anti-patterns

### Navigation.md
- NavigationStack (modern approach)
- NavigationCoordinator pattern
- Type-safe routing
- Sheets and full-screen covers
- Deep linking support
- Tab-based navigation

### Testing.md
- Swift Testing framework (@Test)
- Async/await testing
- Parameterized tests
- ViewModel testing patterns
- Mock service creation

### UIComponents.md
- LoadingView, EmptyStateView, ErrorView
- Custom button styles
- Reusable containers (CardView)
- Avatar, Badge, Rating components
- Accessibility support

### AgentInstructions.md
- Skill selection matrix
- Code generation guidelines
- Pattern recognition rules
- Common task templates
- Integration checklist

---

## 🔍 Quick Reference

| Need to... | Check this skill |
|-----------|-----------------|
| Fetch data from API | [NetworkService.md](Skills/NetworkService.md) |
| Build a list screen | [MVVMPattern.md](Skills/MVVMPattern.md) |
| Pass data between views | [StateManagement.md](Skills/StateManagement.md) |
| Navigate to another screen | [Navigation.md](Skills/Navigation.md) |
| Write tests | [Testing.md](Skills/Testing.md) |
| Show loading/error states | [UIComponents.md](Skills/UIComponents.md) |
| Understand property wrappers | [StateManagement.md](Skills/StateManagement.md) |
| Handle deep links | [Navigation.md](Skills/Navigation.md) |
| Create reusable components | [UIComponents.md](Skills/UIComponents.md) |
| AI agent guidance | [AgentInstructions.md](Skills/AgentInstructions.md) |

---

## 🤝 Contributing

Contributions are welcome! If you have:
- Improvements to existing patterns
- New skill suggestions
- Better examples
- Bug fixes in code snippets

Please open an issue or submit a pull request.

---

## 📞 Support

- **Issues**: [GitHub Issues](https://github.com/filipkjamilov/Swift-UI-Agent-Toolkit/issues)
- **Discussions**: [GitHub Discussions](https://github.com/filipkjamilov/Swift-UI-Agent-Toolkit/discussions)

---

**Built for the future of AI-assisted development** 🚀
