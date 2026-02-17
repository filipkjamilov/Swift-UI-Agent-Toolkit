# State Management

Comprehensive guide to SwiftUI property wrappers and state management patterns.

---

## Property Wrapper Decision Tree

```
Is the data coming from outside the view?
├─ YES → Is it an ObservableObject?
│         ├─ YES → Is the parent creating it?
│         │         ├─ YES → Use @ObservedObject
│         │         └─ NO → Use @EnvironmentObject
│         └─ NO → Is it mutable?
│                   ├─ YES → Use @Binding
│                   └─ NO → Just pass as parameter
│
└─ NO → Is it an ObservableObject?
          ├─ YES → Use @StateObject
          └─ NO → Use @State
```

---

## @State

**Purpose**: View-local, simple value types

**Ownership**: View owns the data

**When to use**:
- Simple values (Int, String, Bool, etc.)
- Data that's private to one view
- Temporary UI state (isExpanded, selectedTab, etc.)

```swift
struct CounterView: View {
    @State private var count = 0
    @State private var isShowingDetail = false
    
    var body: some View {
        VStack {
            Text("Count: \(count)")
            Button("Increment") {
                count += 1
            }
            Button("Show Detail") {
                isShowingDetail = true
            }
        }
        .sheet(isPresented: $isShowingDetail) {
            DetailView()
        }
    }
}
```

**Key points**:
- Always mark as `private`
- Never initialize complex objects with @State
- Use for UI-only state

---

## @Binding

**Purpose**: Two-way connection to parent's state

**Ownership**: Parent owns the data

**When to use**:
- Child view needs to read AND write parent's data
- Forms and input controls
- Creating reusable components

```swift
struct ToggleView: View {
    @Binding var isOn: Bool
    
    var body: some View {
        Toggle("Feature Enabled", isOn: $isOn)
    }
}

struct ParentView: View {
    @State private var isFeatureEnabled = false
    
    var body: some View {
        VStack {
            ToggleView(isOn: $isFeatureEnabled)
            
            if isFeatureEnabled {
                Text("Feature is ON")
            }
        }
    }
}
```

**Key points**:
- Use `$` to pass binding
- Child can modify parent's state
- Great for reusable components

---

## @StateObject

**Purpose**: Create and own an ObservableObject

**Ownership**: View owns the object

**When to use**:
- View creates the ViewModel
- Object should persist across view updates
- Need to observe changes to object properties

```swift
@MainActor
class TimerViewModel: ObservableObject {
    @Published var secondsElapsed = 0
    @Published var isRunning = false
    
    private var timer: Timer?
    
    func start() {
        guard !isRunning else { return }
        isRunning = true
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
            self?.secondsElapsed += 1
        }
    }
    
    func stop() {
        timer?.invalidate()
        timer = nil
        isRunning = false
    }
}

struct TimerView: View {
    @StateObject private var viewModel = TimerViewModel()
    
    var body: some View {
        VStack {
            Text("\(viewModel.secondsElapsed)s")
                .font(.largeTitle)
            
            Button(viewModel.isRunning ? "Stop" : "Start") {
                if viewModel.isRunning {
                    viewModel.stop()
                } else {
                    viewModel.start()
                }
            }
        }
    }
}
```

**Key points**:
- Creates object once, persists across view updates
- Use when THIS view creates the object
- Mark as `private` when possible

---

## @ObservedObject

**Purpose**: Observe an ObservableObject from parent

**Ownership**: Parent owns the object

**When to use**:
- ViewModel is passed from parent
- Child view observes changes
- Object is created elsewhere

```swift
struct ChildView: View {
    @ObservedObject var viewModel: TimerViewModel
    
    var body: some View {
        Text("\(viewModel.secondsElapsed)s")
    }
}

struct ParentView: View {
    @StateObject private var viewModel = TimerViewModel()
    
    var body: some View {
        VStack {
            ChildView(viewModel: viewModel)
            
            Button("Start") {
                viewModel.start()
            }
        }
    }
}
```

**Key points**:
- Don't create the object here
- Parent must use @StateObject
- View updates when @Published properties change

---

## @EnvironmentObject

**Purpose**: Share objects across view hierarchy

**Ownership**: Ancestor provides the object

**When to use**:
- App-wide state (user session, settings, theme)
- Deeply nested views need access
- Avoid prop drilling

```swift
class AppSettings: ObservableObject {
    @Published var isDarkMode = false
    @Published var fontSize: Double = 16
}

struct RootView: View {
    @StateObject private var settings = AppSettings()
    
    var body: some View {
        NavigationStack {
            ContentView()
                .environmentObject(settings)
        }
    }
}

struct ContentView: View {
    var body: some View {
        VStack {
            SettingsView()
            ProfileView()
        }
    }
}

struct SettingsView: View {
    @EnvironmentObject private var settings: AppSettings
    
    var body: some View {
        Toggle("Dark Mode", isOn: $settings.isDarkMode)
    }
}

struct ProfileView: View {
    @EnvironmentObject private var settings: AppSettings
    
    var body: some View {
        Text("User Profile")
            .font(.system(size: settings.fontSize))
    }
}
```

**Key points**:
- Must call `.environmentObject()` in ancestor
- Crashes if not provided
- Good for app-wide state
- Avoid overuse (makes dependencies unclear)

---

## @Environment

**Purpose**: Read system or custom environment values

**When to use**:
- System values (dismiss, colorScheme, etc.)
- Custom environment values
- Read-only values from environment

```swift
struct DetailView: View {
    @Environment(\.dismiss) private var dismiss
    @Environment(\.colorScheme) private var colorScheme
    
    var body: some View {
        VStack {
            Text("Current scheme: \(colorScheme == .dark ? "Dark" : "Light")")
            
            Button("Close") {
                dismiss()
            }
        }
    }
}
```

### Custom Environment Values

```swift
// Define key
private struct ThemeKey: EnvironmentKey {
    static let defaultValue = Theme.default
}

// Extend EnvironmentValues
extension EnvironmentValues {
    var appTheme: Theme {
        get { self[ThemeKey.self] }
        set { self[ThemeKey.self] = newValue }
    }
}

struct Theme {
    let primaryColor: Color
    let secondaryColor: Color
    
    static let `default` = Theme(primaryColor: .blue, secondaryColor: .gray)
}

// Set value
SomeView()
    .environment(\.appTheme, Theme.default)

// Read value
struct SomeView: View {
    @Environment(\.appTheme) private var theme
    
    var body: some View {
        Text("Hello")
            .foregroundStyle(theme.primaryColor)
    }
}
```

---

## Common System Environment Values

```swift
@Environment(\.dismiss) private var dismiss                    // Dismiss current presentation
@Environment(\.colorScheme) private var colorScheme            // Light/Dark mode
@Environment(\.horizontalSizeClass) private var sizeClass      // Compact/Regular
@Environment(\.openURL) private var openURL                    // Open URLs
@Environment(\.scenePhase) private var scenePhase              // App lifecycle
@Environment(\.dynamicTypeSize) private var dynamicTypeSize    // Text size
@Environment(\.isEnabled) private var isEnabled                // View enabled state
```

---

## When to Use Which?

| Scenario | Property Wrapper |
|----------|-----------------|
| Simple counter in a view | @State |
| Form field values | @State |
| Toggle boolean | @State |
| Sheet presentation | @State |
| Two-way binding to child | @Binding |
| Custom text field component | @Binding |
| Creating a ViewModel | @StateObject |
| Receiving ViewModel from parent | @ObservedObject |
| App-wide settings | @EnvironmentObject |
| User session state | @EnvironmentObject |
| Dismiss action | @Environment(\.dismiss) |
| Color scheme | @Environment(\.colorScheme) |

---

## Best Practices

### 1. Always Use Private

```swift
// Good
@State private var count = 0
@StateObject private var viewModel = ViewModel()

// Avoid (unless you have a specific reason)
@State var count = 0
```

### 2. Never Force Unwrap

```swift
// Bad
@EnvironmentObject var settings: AppSettings
// If not provided, app crashes!

// Better - provide default or handle gracefully
@StateObject private var settings = AppSettings()
```

### 3. Use Computed Properties for Derived State

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var items: [Item] = []
    @Published var searchQuery = ""
    
    // Good: Computed property
    var filteredItems: [Item] {
        guard !searchQuery.isEmpty else { return items }
        return items.filter { $0.name.contains(searchQuery) }
    }
}
```

### 4. Mark ObservableObject with @MainActor

```swift
// Good
@MainActor
class ViewModel: ObservableObject {
    @Published var data: [Item] = []
}

// Ensures UI updates on main thread
```

### 5. Don't Overuse @Published

```swift
class ViewModel: ObservableObject {
    @Published var items: [Item] = []          // ✅ UI needs this
    @Published var isLoading = false           // ✅ UI needs this
    
    private var cache: [String: Data] = [:]    // ✅ Internal only, not @Published
}
```

---

## Common Patterns

### Loading State

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var data: [Item] = []
    @Published var isLoading = false
    
    func loadData() async {
        isLoading = true
        defer { isLoading = false }
        
        // Fetch data
    }
}
```

### Error Handling

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var errorMessage: String?
    
    func performAction() async {
        do {
            // Action
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}
```

### Form State

```swift
struct FormView: View {
    @State private var name = ""
    @State private var email = ""
    @State private var isValid: Bool {
        !name.isEmpty && email.contains("@")
    }
    
    var body: some View {
        Form {
            TextField("Name", text: $name)
            TextField("Email", text: $email)
            
            Button("Submit") {
                // Submit
            }
            .disabled(!isValid)
        }
    }
}
```

---

## Anti-Patterns

❌ **Don't use @State for complex objects**
```swift
// Bad
@State private var viewModel = ViewModel()

// Good
@StateObject private var viewModel = ViewModel()
```

❌ **Don't use @StateObject for passed objects**
```swift
// Bad
struct ChildView: View {
    @StateObject var viewModel: ViewModel  // View doesn't create it!
}

// Good
struct ChildView: View {
    @ObservedObject var viewModel: ViewModel
}
```

❌ **Don't pass @EnvironmentObject everywhere**
```swift
// Bad - makes dependencies unclear
struct View1: View {
    @EnvironmentObject var state: AppState
}
struct View2: View {
    @EnvironmentObject var state: AppState
}
// Lots of views...

// Better - explicit dependencies
struct View1: View {
    let state: AppState
}
```

---

## Key Takeaways

- ✅ Use @State for simple, view-local values
- ✅ Use @Binding for two-way parent-child communication
- ✅ Use @StateObject when creating ObservableObjects
- ✅ Use @ObservedObject when receiving ObservableObjects
- ✅ Use @EnvironmentObject for app-wide state (sparingly)
- ✅ Use @Environment for system and custom environment values
- ✅ Always mark as private when possible
- ✅ Mark ViewModels with @MainActor
