# Navigation Patterns

Complex navigation in SwiftUI using NavigationStack, sheets, and modals.

---

## Navigation Types

1. **Stack Navigation** - Hierarchical push/pop (NavigationStack)
2. **Sheet** - Modal presentation, dismissible
3. **Full Screen Cover** - Immersive modal, full screen
4. **Tab Navigation** - Parallel app sections

---

## Pattern: NavigationCoordinator

Centralized navigation management for testability and maintainability.

### 1. Define Routes

```swift
enum AppRoute: Hashable {
    case profile(user: User)
    case settings
    case article(id: String)
    case editProfile(user: User)
}

struct User: Hashable, Identifiable {
    let id: String
    let name: String
    let email: String
}
```

### 2. Create Coordinator

```swift
@MainActor
class NavigationCoordinator: ObservableObject {
    @Published var path = NavigationPath()
    @Published var presentedSheet: SheetDestination?
    @Published var presentedFullScreen: FullScreenDestination?
    
    enum SheetDestination: Identifiable {
        case createPost
        case filter
        case share(content: String)
        
        var id: String {
            switch self {
            case .createPost: return "createPost"
            case .filter: return "filter"
            case .share: return "share"
            }
        }
    }
    
    enum FullScreenDestination: Identifiable {
        case onboarding
        case camera
        
        var id: String {
            switch self {
            case .onboarding: return "onboarding"
            case .camera: return "camera"
            }
        }
    }
    
    // Navigation actions
    func navigate(to route: AppRoute) {
        path.append(route)
    }
    
    func navigateBack() {
        path.removeLast()
    }
    
    func navigateToRoot() {
        path.removeLast(path.count)
    }
    
    func presentSheet(_ sheet: SheetDestination) {
        presentedSheet = sheet
    }
    
    func presentFullScreen(_ destination: FullScreenDestination) {
        presentedFullScreen = destination
    }
}
```

### 3. Set Up NavigationStack

```swift
struct RootView: View {
    @StateObject private var coordinator = NavigationCoordinator()
    
    var body: some View {
        NavigationStack(path: $coordinator.path) {
            HomeView()
                .navigationDestination(for: AppRoute.self) { route in
                    destinationView(for: route)
                }
        }
        .environmentObject(coordinator)
        .sheet(item: $coordinator.presentedSheet) { sheet in
            sheetView(for: sheet)
        }
        .fullScreenCover(item: $coordinator.presentedFullScreen) { destination in
            fullScreenView(for: destination)
        }
    }
    
    @ViewBuilder
    private func destinationView(for route: AppRoute) -> some View {
        switch route {
        case .profile(let user):
            ProfileView(user: user)
        case .settings:
            SettingsView()
        case .article(let id):
            ArticleDetailView(articleId: id)
        case .editProfile(let user):
            EditProfileView(user: user)
        }
    }
    
    @ViewBuilder
    private func sheetView(for sheet: NavigationCoordinator.SheetDestination) -> some View {
        switch sheet {
        case .createPost:
            CreatePostView()
        case .filter:
            FilterView()
        case .share(let content):
            ShareView(content: content)
        }
    }
    
    @ViewBuilder
    private func fullScreenView(for destination: NavigationCoordinator.FullScreenDestination) -> some View {
        switch destination {
        case .onboarding:
            OnboardingView()
        case .camera:
            CameraView()
        }
    }
}
```

### 4. Navigate from Views

```swift
struct HomeView: View {
    @EnvironmentObject private var coordinator: NavigationCoordinator
    
    var body: some View {
        VStack {
            Button("Go to Profile") {
                let user = User(id: "1", name: "John", email: "john@example.com")
                coordinator.navigate(to: .profile(user: user))
            }
            
            Button("Show Settings") {
                coordinator.navigate(to: .settings)
            }
            
            Button("Create Post (Sheet)") {
                coordinator.presentSheet(.createPost)
            }
        }
        .navigationTitle("Home")
    }
}

struct ProfileView: View {
    @EnvironmentObject private var coordinator: NavigationCoordinator
    let user: User
    
    var body: some View {
        VStack {
            Text(user.name)
            
            Button("Edit Profile") {
                coordinator.navigate(to: .editProfile(user: user))
            }
            
            Button("Back to Root") {
                coordinator.navigateToRoot()
            }
        }
        .navigationTitle("Profile")
    }
}
```

---

## Basic NavigationStack

For simple hierarchical navigation:

```swift
struct SimpleNavigationExample: View {
    var body: some View {
        NavigationStack {
            List {
                NavigationLink("Profile") {
                    ProfileView()
                }
                
                NavigationLink("Settings") {
                    SettingsView()
                }
            }
            .navigationTitle("Home")
        }
    }
}
```

---

## Sheet Presentation

For modal, dismissible content:

```swift
struct SheetExample: View {
    @State private var isShowingSheet = false
    
    var body: some View {
        Button("Show Sheet") {
            isShowingSheet = true
        }
        .sheet(isPresented: $isShowingSheet) {
            SheetContent()
        }
    }
}

struct SheetContent: View {
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        NavigationStack {
            VStack {
                Text("Sheet Content")
            }
            .navigationTitle("Sheet")
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("Cancel") {
                        dismiss()
                    }
                }
            }
        }
    }
}
```

---

## Full Screen Cover

For immersive, full-screen experiences:

```swift
struct FullScreenExample: View {
    @State private var isShowingFullScreen = false
    
    var body: some View {
        Button("Show Full Screen") {
            isShowingFullScreen = true
        }
        .fullScreenCover(isPresented: $isShowingFullScreen) {
            FullScreenContent()
        }
    }
}
```

---

## Tab Navigation

Each tab has its own NavigationStack:

```swift
struct TabNavigationExample: View {
    @State private var selectedTab = 0
    
    var body: some View {
        TabView(selection: $selectedTab) {
            NavigationStack {
                HomeTabView()
            }
            .tabItem {
                Label("Home", systemImage: "house")
            }
            .tag(0)
            
            NavigationStack {
                SearchTabView()
            }
            .tabItem {
                Label("Search", systemImage: "magnifyingglass")
            }
            .tag(1)
            
            NavigationStack {
                ProfileTabView()
            }
            .tabItem {
                Label("Profile", systemImage: "person")
            }
            .tag(2)
        }
    }
}
```

---

## Deep Linking

Handle URLs and navigate programmatically:

```swift
struct DeepLinkingExample: View {
    @StateObject private var coordinator = NavigationCoordinator()
    
    var body: some View {
        NavigationStack(path: $coordinator.path) {
            HomeView()
                .navigationDestination(for: AppRoute.self) { route in
                    destinationView(for: route)
                }
        }
        .environmentObject(coordinator)
        .onOpenURL { url in
            handleDeepLink(url: url)
        }
    }
    
    private func handleDeepLink(url: URL) {
        // Parse URL: myapp://article/123
        if url.pathComponents.contains("article"),
           let articleId = url.pathComponents.last {
            coordinator.navigateToRoot()
            coordinator.navigate(to: .article(id: articleId))
        }
    }
}
```

---

## Best Practices

### 1. Use NavigationStack (not NavigationView)

```swift
// ✅ Good
NavigationStack {
    ContentView()
}

// ❌ Deprecated
NavigationView {
    ContentView()
}
```

### 2. One NavigationStack Per Tab

```swift
// ✅ Good
TabView {
    NavigationStack { Tab1() }
    NavigationStack { Tab2() }
}

// ❌ Bad - nested stacks
NavigationStack {
    TabView {
        Tab1()
        Tab2()
    }
}
```

### 3. Never Nest NavigationStacks

```swift
// ❌ NEVER DO THIS
NavigationStack {
    NavigationStack {
        ContentView()
    }
}
```

### 4. Type-Safe Routes

```swift
// ✅ Good - Compiler catches errors
enum AppRoute: Hashable {
    case profile(id: String)
    case settings
}

// ❌ Bad - Typos possible
func navigate(to: String) {
    // "proffile" vs "profile"
}
```

### 5. Centralize Navigation

```swift
// ✅ Good - Testable, maintainable
class NavigationCoordinator: ObservableObject {
    func navigate(to route: AppRoute) {
        // Centralized logic
    }
}

// ❌ Bad - Scattered navigation logic
Button {
    path.append(SomeRoute.detail)
}
```

### 6. Use Environment Dismiss

```swift
// ✅ Good
@Environment(\.dismiss) private var dismiss

Button("Close") {
    dismiss()
}

// ❌ Avoid - less flexible
@Binding var isPresented: Bool

Button("Close") {
    isPresented = false
}
```

---

## Common Patterns

### Form in Sheet

```swift
struct FormSheetView: View {
    @Environment(\.dismiss) private var dismiss
    @State private var name = ""
    @State private var email = ""
    
    var isValid: Bool {
        !name.isEmpty && email.contains("@")
    }
    
    var body: some View {
        NavigationStack {
            Form {
                TextField("Name", text: $name)
                TextField("Email", text: $email)
            }
            .navigationTitle("New Contact")
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("Cancel") { dismiss() }
                }
                ToolbarItem(placement: .confirmationAction) {
                    Button("Save") {
                        // Save and dismiss
                        dismiss()
                    }
                    .disabled(!isValid)
                }
            }
        }
    }
}
```

### Navigation with Data Passing

```swift
NavigationLink(value: user) {
    UserRow(user: user)
}
.navigationDestination(for: User.self) { user in
    UserDetailView(user: user)
}
```

### Programmatic Navigation

```swift
@State private var path = NavigationPath()

Button("Go Deep") {
    path.append(Route.level1)
    path.append(Route.level2)
    path.append(Route.level3)
}

Button("Back to Root") {
    path.removeLast(path.count)
}
```

---

## Key Takeaways

- ✅ Use NavigationStack, not NavigationView
- ✅ One NavigationStack per tab in TabView
- ✅ Never nest NavigationStacks
- ✅ Use type-safe routes with Hashable enums
- ✅ Centralize navigation in a Coordinator
- ✅ Use @Environment(\.dismiss) in modals
- ✅ Handle deep links with .onOpenURL
- ✅ Test navigation by mocking coordinator
