# Testing with Swift Testing Framework

Modern unit testing using Swift Testing framework (not XCTest).

---

## When to Use

- Writing unit tests for business logic
- Testing ViewModels
- Testing service layers
- Parameterized testing (multiple inputs)
- Async/await testing

---

## Basic Test Structure

```swift
import Testing
@testable import YourApp

struct CalculatorTests {
    @Test("Addition works correctly")
    func testAddition() {
        let result = 2 + 3
        #expect(result == 5)
    }
    
    @Test("Subtraction works correctly")
    func testSubtraction() {
        let result = 10 - 4
        #expect(result == 6)
    }
}
```

---

## Key Concepts

### @Test Macro

```swift
@Test("Description of what this tests")
func testSomething() {
    // Test code
}

// Or without description
@Test func testFeature() {
    // Test code
}
```

### #expect Macro

```swift
#expect(value == expected)              // Assert equality
#expect(items.count > 0)                // Assert condition
#expect(result != nil)                  // Assert not nil
#expect(throws: SomeError.self) {       // Assert throws error
    try dangerousOperation()
}
```

---

## Testing ViewModels

```swift
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

// Tests
struct UserListViewModelTests {
    @Test("Successfully loads users")
    @MainActor
    func testLoadUsersSuccess() async throws {
        let mockService = MockUserService()
        mockService.mockUsers = [
            User(id: "1", name: "John", email: "john@example.com"),
            User(id: "2", name: "Jane", email: "jane@example.com")
        ]
        
        let viewModel = UserListViewModel(service: mockService)
        
        await viewModel.loadUsers()
        
        #expect(viewModel.users.count == 2)
        #expect(viewModel.isLoading == false)
        #expect(viewModel.errorMessage == nil)
    }
    
    @Test("Handles errors correctly")
    @MainActor
    func testLoadUsersError() async throws {
        let mockService = MockUserService()
        mockService.shouldFail = true
        
        let viewModel = UserListViewModel(service: mockService)
        
        await viewModel.loadUsers()
        
        #expect(viewModel.users.isEmpty)
        #expect(viewModel.errorMessage != nil)
    }
}
```

---

## Mock Services

```swift
protocol UserServiceProtocol {
    func fetchUsers() async throws -> [User]
}

class MockUserService: UserServiceProtocol {
    var shouldFail = false
    var mockUsers: [User] = []
    
    func fetchUsers() async throws -> [User] {
        if shouldFail {
            throw NSError(domain: "TestError", code: 500, userInfo: [
                NSLocalizedDescriptionKey: "Mock error"
            ])
        }
        
        // Simulate network delay (optional)
        try await Task.sleep(nanoseconds: 100_000_000)
        
        return mockUsers
    }
}
```

---

## Async/Await Testing

```swift
struct AsyncTests {
    @Test("Async data fetch succeeds")
    func testAsyncFetch() async throws {
        let service = DataService()
        let data = try await service.fetchData()
        
        #expect(data.count > 0)
    }
    
    @Test("Async error handling works")
    func testAsyncError() async {
        let service = FailingDataService()
        
        await #expect(throws: DataServiceError.networkError) {
            try await service.fetchData()
        }
    }
}
```

---

## Parameterized Tests

Test multiple inputs with one test function:

```swift
struct ParameterizedTests {
    @Test("Addition with multiple values", arguments: [
        (2, 3, 5),
        (0, 0, 0),
        (-5, 3, -2),
        (100, 200, 300)
    ])
    func testAddition(a: Int, b: Int, expected: Int) {
        let result = a + b
        #expect(result == expected)
    }
    
    @Test("Email validation", arguments: [
        ("user@example.com", true),
        ("invalid.email", false),
        ("@example.com", false),
        ("user@", false)
    ])
    func testEmailValidation(email: String, isValid: Bool) {
        let result = validateEmail(email)
        #expect(result == isValid)
    }
}
```

---

## Test Suites

Organize related tests:

```swift
@Suite("User Management")
struct UserManagementTests {
    @Suite("User Creation")
    struct UserCreationTests {
        @Test func validUserCreation() {
            // Test
        }
        
        @Test func invalidEmailRejected() {
            // Test
        }
    }
    
    @Suite("User Deletion")
    struct UserDeletionTests {
        @Test func userDeletion() {
            // Test
        }
    }
}
```

---

## Test Traits

### Disable Tests

```swift
@Test(.disabled("Temporarily disabled due to API changes"))
func testFeature() {
    // Test code
}
```

### Conditional Execution

```swift
@Test(.enabled(if: ProcessInfo.processInfo.environment["RUN_SLOW_TESTS"] != nil))
func slowTest() async throws {
    // Slow test
}
```

### Time Limits

```swift
@Test(.timeLimit(.seconds(5)))
func testPerformance() {
    // Must complete within 5 seconds
}
```

---

## Testing Patterns

### Arrange-Act-Assert

```swift
@Test("User profile updates successfully")
@MainActor
func testProfileUpdate() async throws {
    // Arrange
    let mockService = MockUserService()
    let viewModel = ProfileViewModel(service: mockService)
    let newName = "Updated Name"
    
    // Act
    await viewModel.updateProfile(name: newName)
    
    // Assert
    #expect(viewModel.profile?.name == newName)
    #expect(viewModel.errorMessage == nil)
}
```

### Given-When-Then

```swift
@Test("Shopping cart calculates total correctly")
func testCartTotal() {
    // Given
    let cart = ShoppingCart()
    cart.addItem(Item(name: "Book", price: 10.0))
    cart.addItem(Item(name: "Pen", price: 2.0))
    
    // When
    let total = cart.calculateTotal()
    
    // Then
    #expect(total == 12.0)
}
```

---

## Common Testing Scenarios

### Testing Loading States

```swift
@Test("Loading state is set correctly")
@MainActor
func testLoadingState() async throws {
    let viewModel = DataViewModel(service: MockService())
    
    // Before loading
    #expect(viewModel.isLoading == false)
    
    // Start loading
    Task {
        await viewModel.loadData()
    }
    
    // During loading (check immediately)
    try await Task.sleep(nanoseconds: 10_000_000)
    #expect(viewModel.isLoading == true)
    
    // Wait for completion
    try await Task.sleep(nanoseconds: 200_000_000)
    #expect(viewModel.isLoading == false)
}
```

### Testing Search/Filter

```swift
@Test("Search filters results correctly")
@MainActor
func testSearch() async throws {
    let mockService = MockArticleService()
    mockService.mockArticles = [
        Article(id: UUID(), title: "Swift Programming", author: "John"),
        Article(id: UUID(), title: "Kotlin Guide", author: "Jane"),
        Article(id: UUID(), title: "SwiftUI Basics", author: "Bob")
    ]
    
    let viewModel = ArticleListViewModel(service: mockService)
    await viewModel.loadArticles()
    
    viewModel.searchQuery = "Swift"
    
    #expect(viewModel.filteredArticles.count == 2)
    #expect(viewModel.filteredArticles.allSatisfy { $0.title.contains("Swift") })
}
```

### Testing Error Handling

```swift
@Test("Network error is handled correctly")
@MainActor
func testNetworkError() async throws {
    let mockService = MockService()
    mockService.shouldFail = true
    
    let viewModel = DataViewModel(service: mockService)
    await viewModel.loadData()
    
    #expect(viewModel.data.isEmpty)
    #expect(viewModel.errorMessage != nil)
    #expect(viewModel.errorMessage?.contains("error") == true)
}
```

---

## Best Practices

### 1. Use Descriptive Test Names

```swift
// ✅ Good
@Test("User login succeeds with valid credentials")

// ❌ Bad
@Test func test1()
```

### 2. Test One Thing Per Test

```swift
// ✅ Good
@Test("Addition works") {
    #expect(2 + 2 == 4)
}

@Test("Subtraction works") {
    #expect(5 - 3 == 2)
}

// ❌ Bad - testing multiple things
@Test("Math operations") {
    #expect(2 + 2 == 4)
    #expect(5 - 3 == 2)
    #expect(3 * 4 == 12)
}
```

### 3. Use Mock Services

```swift
// ✅ Good - Testable
class ViewModel {
    private let service: ServiceProtocol
    
    init(service: ServiceProtocol) {
        self.service = service
    }
}

// Test with MockService
let viewModel = ViewModel(service: MockService())

// ❌ Bad - Hard to test
class ViewModel {
    private let service = RealService() // Hardcoded!
}
```

### 4. Test Edge Cases

```swift
@Test("Empty array returns zero sum")
func testEmptyArray() {
    let sum = calculateSum([])
    #expect(sum == 0)
}

@Test("Nil value is handled safely")
func testNilHandling() {
    let result = process(nil)
    #expect(result == defaultValue)
}

@Test("Large numbers don't overflow")
func testLargeNumbers() {
    let result = Int.max + 0
    #expect(result == Int.max)
}
```

### 5. Keep Tests Independent

```swift
// ✅ Good - Each test is independent
struct Tests {
    @Test func test1() {
        let data = createData()
        // Use data
    }
    
    @Test func test2() {
        let data = createData()
        // Use data
    }
}

// ❌ Bad - Tests depend on order
struct Tests {
    var sharedData: [Item] = []
    
    @Test func test1() {
        sharedData.append(item) // Modifies shared state!
    }
    
    @Test func test2() {
        // Depends on test1 running first
        #expect(sharedData.count == 1)
    }
}
```

### 6. Mark Async Tests with await

```swift
// ✅ Good
@Test func testAsync() async throws {
    let result = try await service.fetch()
    #expect(result != nil)
}

// ❌ Bad - Forgetting await
@Test func testAsync() throws {
    let result = service.fetch() // Won't wait!
    #expect(result != nil)
}
```

---

## XCTest Migration

If migrating from XCTest:

| XCTest | Swift Testing |
|--------|---------------|
| `XCTAssertEqual(a, b)` | `#expect(a == b)` |
| `XCTAssertTrue(condition)` | `#expect(condition)` |
| `XCTAssertNil(value)` | `#expect(value == nil)` |
| `XCTAssertThrowsError` | `#expect(throws: Error.self) { }` |
| `func testName()` | `@Test func testName()` |
| `class TestClass: XCTestCase` | `struct TestStruct` |

---

## Key Takeaways

- ✅ Use @Test macro for test functions
- ✅ Use #expect for assertions
- ✅ Mark ViewModels with @MainActor in tests
- ✅ Use async/await for async tests
- ✅ Create mock services for dependency injection
- ✅ Test one thing per test function
- ✅ Use descriptive test names
- ✅ Test edge cases and error scenarios
- ✅ Keep tests independent
- ✅ Use @Suite to organize related tests
