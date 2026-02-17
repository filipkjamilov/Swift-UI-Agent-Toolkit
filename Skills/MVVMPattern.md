# MVVM Pattern

Complete Model-View-ViewModel architecture for SwiftUI applications.

---

## When to Use

- Building feature-complete screens
- Need clear separation of concerns
- Want testable business logic
- Creating list-based interfaces
- Any non-trivial SwiftUI view

---

## Architecture Overview

```
Model (Data)
    ↓
Service (Data Source)
    ↓
ViewModel (Business Logic)
    ↓
View (UI)
```

---

## Implementation Pattern

### 1. Model

Define your data structures:

```swift
struct Article: Identifiable, Codable {
    let id: UUID
    let title: String
    let author: String
    let content: String
    let publishedDate: Date
    let imageURL: String?
}
```

### 2. Service Protocol

Define data source interface:

```swift
protocol ArticleServiceProtocol {
    func fetchArticles() async throws -> [Article]
    func fetchArticle(id: UUID) async throws -> Article
    func createArticle(_ article: Article) async throws -> Article
}
```

### 3. Mock Service (for development/testing)

```swift
class MockArticleService: ArticleServiceProtocol {
    func fetchArticles() async throws -> [Article] {
        try await Task.sleep(nanoseconds: 1_000_000_000)
        return [
            Article(id: UUID(), title: "Test Article", author: "Author", content: "Content", publishedDate: Date()),
        ]
    }
    
    func fetchArticle(id: UUID) async throws -> Article {
        // Return mock article
    }
    
    func createArticle(_ article: Article) async throws -> Article {
        // Return created article
    }
}
```

### 4. ViewModel

Business logic and state management:

```swift
@MainActor
class ArticleListViewModel: ObservableObject {
    // Published state
    @Published var articles: [Article] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    @Published var searchQuery = ""
    
    // Dependencies
    private let service: ArticleServiceProtocol
    
    // Computed properties
    var filteredArticles: [Article] {
        guard !searchQuery.isEmpty else { return articles }
        return articles.filter { article in
            article.title.localizedCaseInsensitiveContains(searchQuery) ||
            article.author.localizedCaseInsensitiveContains(searchQuery)
        }
    }
    
    // Initialization with dependency injection
    init(service: ArticleServiceProtocol = MockArticleService()) {
        self.service = service
    }
    
    // Actions
    func loadArticles() async {
        isLoading = true
        errorMessage = nil
        defer { isLoading = false }
        
        do {
            articles = try await service.fetchArticles()
        } catch {
            errorMessage = "Failed to load: \(error.localizedDescription)"
        }
    }
    
    func refreshArticles() async {
        await loadArticles()
    }
    
    func createArticle(title: String, author: String, content: String) async {
        let newArticle = Article(
            id: UUID(),
            title: title,
            author: author,
            content: content,
            publishedDate: Date(),
            imageURL: nil
        )
        
        do {
            let created = try await service.createArticle(newArticle)
            articles.insert(created, at: 0)
        } catch {
            errorMessage = "Failed to create: \(error.localizedDescription)"
        }
    }
}
```

### 5. View

UI presentation:

```swift
struct ArticleListView: View {
    @StateObject private var viewModel = ArticleListViewModel()
    @State private var showingCreateSheet = false
    
    var body: some View {
        NavigationStack {
            ZStack {
                if viewModel.isLoading && viewModel.articles.isEmpty {
                    ProgressView("Loading articles...")
                } else {
                    articleList
                }
            }
            .navigationTitle("Articles")
            .toolbar {
                ToolbarItem(placement: .primaryAction) {
                    Button {
                        showingCreateSheet = true
                    } label: {
                        Image(systemName: "plus")
                    }
                }
            }
            .searchable(text: $viewModel.searchQuery, prompt: "Search articles")
            .refreshable {
                await viewModel.refreshArticles()
            }
            .sheet(isPresented: $showingCreateSheet) {
                CreateArticleView(viewModel: viewModel)
            }
            .task {
                await viewModel.loadArticles()
            }
            .alert("Error", isPresented: .constant(viewModel.errorMessage != nil)) {
                Button("OK") {
                    viewModel.errorMessage = nil
                }
            } message: {
                if let errorMessage = viewModel.errorMessage {
                    Text(errorMessage)
                }
            }
        }
    }
    
    private var articleList: some View {
        List(viewModel.filteredArticles) { article in
            NavigationLink {
                ArticleDetailView(article: article)
            } label: {
                VStack(alignment: .leading, spacing: 8) {
                    Text(article.title)
                        .font(.headline)
                    Text("By \(article.author)")
                        .font(.subheadline)
                        .foregroundStyle(.secondary)
                }
            }
        }
        .overlay {
            if viewModel.filteredArticles.isEmpty {
                ContentUnavailableView.search
            }
        }
    }
}
```

---

## Key Principles

### Separation of Concerns

- **Model**: Data structures only, no logic
- **Service**: Data fetching/persistence, no UI
- **ViewModel**: Business logic, no UI code
- **View**: UI presentation, minimal logic

### State Management

- Use `@Published` for state that affects UI
- Use `@StateObject` to create and own ViewModel
- Use `@ObservedObject` when ViewModel is passed from parent
- Keep state in ViewModel, not in View

### Dependency Injection

```swift
// Good: Inject dependencies
init(service: ArticleServiceProtocol) {
    self.service = service
}

// Bad: Hard-coded dependencies
init() {
    self.service = ArticleService()
}
```

### Async Operations

```swift
// Always use async/await
func loadData() async {
    isLoading = true
    defer { isLoading = false }
    
    do {
        data = try await service.fetchData()
    } catch {
        errorMessage = error.localizedDescription
    }
}
```

---

## Common Patterns

### Loading States

```swift
@Published var isLoading = false

func loadData() async {
    isLoading = true
    defer { isLoading = false }
    // Fetch data
}

// In View
if viewModel.isLoading {
    ProgressView()
}
```

### Error Handling

```swift
@Published var errorMessage: String?

do {
    data = try await service.fetch()
} catch {
    errorMessage = error.localizedDescription
}

// In View
.alert("Error", isPresented: .constant(viewModel.errorMessage != nil)) {
    Button("OK") { viewModel.errorMessage = nil }
} message: {
    Text(viewModel.errorMessage ?? "")
}
```

### Search/Filtering

```swift
@Published var searchQuery = ""

var filteredItems: [Item] {
    guard !searchQuery.isEmpty else { return items }
    return items.filter { $0.name.contains(searchQuery) }
}

// In View
.searchable(text: $viewModel.searchQuery)
List(viewModel.filteredItems) { item in
    // ...
}
```

### Pull to Refresh

```swift
func refreshData() async {
    await loadData()
}

// In View
.refreshable {
    await viewModel.refreshData()
}
```

---

## Testing Pattern

```swift
import Testing
@testable import YourApp

struct ArticleListViewModelTests {
    @Test("Loads articles successfully")
    @MainActor
    func testLoadArticles() async throws {
        let mockService = MockArticleService()
        let viewModel = ArticleListViewModel(service: mockService)
        
        await viewModel.loadArticles()
        
        #expect(viewModel.articles.count > 0)
        #expect(viewModel.isLoading == false)
        #expect(viewModel.errorMessage == nil)
    }
    
    @Test("Handles errors correctly")
    @MainActor
    func testErrorHandling() async throws {
        let mockService = MockArticleService()
        mockService.shouldFail = true
        let viewModel = ArticleListViewModel(service: mockService)
        
        await viewModel.loadArticles()
        
        #expect(viewModel.articles.isEmpty)
        #expect(viewModel.errorMessage != nil)
    }
    
    @Test("Filters articles by search query")
    @MainActor
    func testSearch() async throws {
        let viewModel = ArticleListViewModel(service: MockArticleService())
        await viewModel.loadArticles()
        
        viewModel.searchQuery = "Swift"
        
        #expect(viewModel.filteredArticles.count <= viewModel.articles.count)
    }
}
```

---

## Best Practices

1. ✅ **Always use protocols** for services (testability)
2. ✅ **Mark ViewModels with @MainActor** (UI updates on main thread)
3. ✅ **Use dependency injection** (pass services to ViewModel)
4. ✅ **Handle loading and error states** (better UX)
5. ✅ **Use computed properties** for derived state (filtering, sorting)
6. ✅ **Keep Views simple** (delegate logic to ViewModel)
7. ✅ **Use defer for cleanup** (isLoading = false)
8. ✅ **Test ViewModels** (business logic should be tested)

---

## Anti-Patterns to Avoid

❌ Don't put business logic in Views
❌ Don't create ViewModels inside ViewModels
❌ Don't use singletons (hurts testability)
❌ Don't ignore error handling
❌ Don't use force unwrapping
❌ Don't mix UI code in ViewModels
