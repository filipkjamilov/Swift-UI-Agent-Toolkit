# Skills Directory

This directory contains skill guides for building Swift and SwiftUI applications. Each skill is a focused markdown document with patterns, examples, and best practices.

---

## Available Skills

### 1. NetworkService.md
**Modern networking with async/await**

- Protocol-based network service architecture
- Generic GET and POST request methods
- Proper error handling with custom error types
- JSON encoding/decoding strategies
- Mock services for testing

**Use when**: Building REST API clients, fetching data from servers

---

### 2. MVVMPattern.md
**Complete MVVM architecture**

- Model-View-ViewModel separation
- Protocol-based service layer
- Observable object state management
- Loading and error states
- Search and filtering patterns

**Use when**: Building any non-trivial feature screen, need testable architecture

---

### 3. StateManagement.md
**SwiftUI property wrappers guide**

- Decision tree for choosing property wrappers
- @State, @Binding, @StateObject, @ObservedObject
- @EnvironmentObject, @Environment
- Custom environment values
- Best practices and anti-patterns

**Use when**: Managing state, passing data between views, need to understand which property wrapper to use

---

### 4. Navigation.md
**Complex navigation patterns**

- NavigationCoordinator pattern
- Type-safe routing with enums
- Stack, sheet, and full-screen navigation
- Tab-based navigation
- Deep linking support

**Use when**: Building multi-screen apps, implementing navigation flows, handling deep links

---

### 5. Testing.md
**Swift Testing framework guide**

- Modern testing with @Test and #expect
- Async/await testing
- Parameterized tests
- ViewModel testing patterns
- Mock service creation

**Use when**: Writing unit tests, testing ViewModels, need to test async code

---

### 6. UIComponents.md
**Reusable UI component patterns**

- LoadingView, EmptyStateView, ErrorView
- Custom button styles
- CardView, AvatarView, BadgeView
- Custom text fields
- Accessibility support

**Use when**: Need consistent UI, building common patterns, creating design system

---

### 7. AgentInstructions.md
**Guidelines for AI coding agents**

- Skill selection matrix
- Code generation guidelines
- Pattern recognition rules
- Common task templates
- Integration checklist

**Use when**: You are an AI agent working on this project, teaching developers the workflow

---

## How to Use These Skills

### For Developers

1. **Reference when building**: Check the relevant skill before implementing a feature
2. **Copy examples**: Use code snippets as starting points
3. **Follow patterns**: Maintain consistency with established practices
4. **Combine skills**: Most features use multiple skills together

### For AI Agents

1. Read `AgentInstructions.md` first for guidance
2. Use the skill selection matrix to choose appropriate patterns
3. Follow code generation guidelines for consistency
4. Combine skills as shown in integration examples

---

## Skill Quick Reference

| Need to... | Read this skill |
|-----------|----------------|
| Fetch data from API | NetworkService.md |
| Build a list screen | MVVMPattern.md |
| Pass data between views | StateManagement.md |
| Navigate to another screen | Navigation.md |
| Write tests | Testing.md |
| Show loading state | UIComponents.md |
| Understand property wrappers | StateManagement.md |
| Handle errors | MVVMPattern.md + UIComponents.md |
| Implement search | MVVMPattern.md |
| Deep link support | Navigation.md |

---

## Integration Example

Most features combine multiple skills:

```swift
// 1. Model (MVVMPattern.md)
struct Article: Identifiable, Codable {
    let id: UUID
    let title: String
    let content: String
}

// 2. Service (NetworkService.md)
protocol ArticleServiceProtocol {
    func fetchArticles() async throws -> [Article]
}

class ArticleService: ArticleServiceProtocol {
    private let networkService: NetworkServiceProtocol
    
    func fetchArticles() async throws -> [Article] {
        try await networkService.fetch(from: "/articles")
    }
}

// 3. ViewModel (MVVMPattern.md + StateManagement.md)
@MainActor
class ArticleListViewModel: ObservableObject {
    @Published var articles: [Article] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private let service: ArticleServiceProtocol
    
    init(service: ArticleServiceProtocol) {
        self.service = service
    }
    
    func loadArticles() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            articles = try await service.fetchArticles()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}

// 4. View (MVVMPattern.md + UIComponents.md + Navigation.md)
struct ArticleListView: View {
    @StateObject private var viewModel = ArticleListViewModel(
        service: ArticleService()
    )
    @EnvironmentObject private var coordinator: NavigationCoordinator
    
    var body: some View {
        Group {
            if viewModel.isLoading {
                LoadingView() // UIComponents.md
            } else if let error = viewModel.error {
                ErrorView(error: error) { // UIComponents.md
                    Task { await viewModel.loadArticles() }
                }
            } else {
                List(viewModel.articles) { article in
                    Button {
                        coordinator.navigate(to: .article(id: article.id)) // Navigation.md
                    } label: {
                        ArticleRow(article: article)
                    }
                }
            }
        }
        .task {
            await viewModel.loadArticles()
        }
    }
}

// 5. Tests (Testing.md)
struct ArticleListViewModelTests {
    @Test("Loads articles successfully")
    @MainActor
    func testLoadArticles() async throws {
        let mockService = MockArticleService()
        let viewModel = ArticleListViewModel(service: mockService)
        
        await viewModel.loadArticles()
        
        #expect(viewModel.articles.count > 0)
        #expect(viewModel.isLoading == false)
    }
}
```

---

## Best Practices Across All Skills

✅ **Modern Swift Concurrency**: Use async/await, not completion handlers  
✅ **Protocol-Oriented**: Protocols for testability and flexibility  
✅ **Type Safety**: Leverage Swift's type system  
✅ **Error Handling**: Proper do-catch and custom error types  
✅ **Separation of Concerns**: Clear boundaries between layers  
✅ **Accessibility**: Support VoiceOver and Dynamic Type  
✅ **Testing**: Designed for testability with dependency injection  
✅ **Documentation**: Clear comments and examples  

---

## Updating Skills

As SwiftUI evolves, update skills to reflect:
- New SwiftUI APIs and patterns
- Updated best practices
- Performance improvements
- Accessibility enhancements
- New platform features

---

## Additional Resources

- **Main guide**: `../CLAUDE.md` - Overall coding standards
- **Apple Documentation**: Use DocumentationSearch in Xcode
- **WWDC Sessions**: https://developer.apple.com/videos/
- **Swift Forums**: https://forums.swift.org

---

**Remember**: These are guides and patterns, not rigid rules. Adapt them to your specific project needs, team standards, and design requirements.
