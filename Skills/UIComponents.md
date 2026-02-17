# Common UI Components

Reusable SwiftUI component patterns for consistent UI.

---

## When to Use

- Need consistent UI across your app
- Building common interface patterns
- Want to avoid repeating UI code
- Creating a design system

---

## Loading View

```swift
struct LoadingView: View {
    let message: String
    
    init(message: String = "Loading...") {
        self.message = message
    }
    
    var body: some View {
        VStack(spacing: 16) {
            ProgressView()
                .scaleEffect(1.5)
            Text(message)
                .foregroundStyle(.secondary)
        }
        .frame(maxWidth: .infinity, maxHeight: .infinity)
    }
}

// Usage
if viewModel.isLoading {
    LoadingView(message: "Fetching data...")
}
```

---

## Empty State View

```swift
struct EmptyStateView: View {
    let systemImage: String
    let title: String
    let subtitle: String?
    let actionTitle: String?
    let action: (() -> Void)?
    
    init(
        systemImage: String = "tray",
        title: String,
        subtitle: String? = nil,
        actionTitle: String? = nil,
        action: (() -> Void)? = nil
    ) {
        self.systemImage = systemImage
        self.title = title
        self.subtitle = subtitle
        self.actionTitle = actionTitle
        self.action = action
    }
    
    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: systemImage)
                .font(.system(size: 60))
                .foregroundStyle(.secondary)
            
            Text(title)
                .font(.title3)
                .fontWeight(.semibold)
            
            if let subtitle {
                Text(subtitle)
                    .font(.subheadline)
                    .foregroundStyle(.secondary)
                    .multilineTextAlignment(.center)
            }
            
            if let actionTitle, let action {
                Button(actionTitle, action: action)
                    .buttonStyle(.borderedProminent)
                    .padding(.top, 8)
            }
        }
        .padding()
    }
}

// Usage
if items.isEmpty {
    EmptyStateView(
        systemImage: "magnifyingglass",
        title: "No results found",
        subtitle: "Try a different search term",
        actionTitle: "Clear Filters",
        action: { viewModel.clearFilters() }
    )
}
```

---

## Error View

```swift
struct ErrorView: View {
    let error: Error
    let retryAction: (() -> Void)?
    
    init(error: Error, retryAction: (() -> Void)? = nil) {
        self.error = error
        self.retryAction = retryAction
    }
    
    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "exclamationmark.triangle")
                .font(.system(size: 60))
                .foregroundStyle(.red)
            
            Text("Something went wrong")
                .font(.title3)
                .fontWeight(.semibold)
            
            Text(error.localizedDescription)
                .font(.subheadline)
                .foregroundStyle(.secondary)
                .multilineTextAlignment(.center)
            
            if let retryAction {
                Button("Try Again", action: retryAction)
                    .buttonStyle(.borderedProminent)
            }
        }
        .padding()
    }
}

// Usage
if let error = viewModel.error {
    ErrorView(error: error) {
        Task { await viewModel.loadData() }
    }
}
```

---

## Custom Button Styles

```swift
struct PrimaryButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .font(.headline)
            .foregroundStyle(.white)
            .padding()
            .frame(maxWidth: .infinity)
            .background(Color.accentColor)
            .cornerRadius(12)
            .scaleEffect(configuration.isPressed ? 0.95 : 1.0)
            .animation(.easeInOut(duration: 0.1), value: configuration.isPressed)
    }
}

struct SecondaryButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .font(.headline)
            .foregroundStyle(Color.accentColor)
            .padding()
            .frame(maxWidth: .infinity)
            .background(Color(.secondarySystemBackground))
            .cornerRadius(12)
            .overlay(
                RoundedRectangle(cornerRadius: 12)
                    .stroke(Color.accentColor, lineWidth: 2)
            )
            .scaleEffect(configuration.isPressed ? 0.95 : 1.0)
    }
}

// Usage
Button("Continue") { }
    .buttonStyle(PrimaryButtonStyle())

Button("Cancel") { }
    .buttonStyle(SecondaryButtonStyle())
```

---

## Card Container

```swift
struct CardView<Content: View>: View {
    let content: Content
    
    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }
    
    var body: some View {
        content
            .padding()
            .background(Color(.secondarySystemBackground))
            .cornerRadius(12)
            .shadow(color: .black.opacity(0.1), radius: 5, x: 0, y: 2)
    }
}

// Usage
CardView {
    VStack(alignment: .leading) {
        Text("Card Title")
            .font(.headline)
        Text("Card content")
            .foregroundStyle(.secondary)
    }
}
```

---

## Avatar View

```swift
struct AvatarView: View {
    let imageURL: String?
    let initials: String
    let size: CGFloat
    
    init(imageURL: String? = nil, initials: String, size: CGFloat = 50) {
        self.imageURL = imageURL
        self.initials = initials
        self.size = size
    }
    
    var body: some View {
        Group {
            if let imageURL, let url = URL(string: imageURL) {
                AsyncImage(url: url) { phase in
                    switch phase {
                    case .success(let image):
                        image
                            .resizable()
                            .scaledToFill()
                    case .empty, .failure:
                        placeholderView
                    @unknown default:
                        placeholderView
                    }
                }
            } else {
                placeholderView
            }
        }
        .frame(width: size, height: size)
        .clipShape(Circle())
    }
    
    private var placeholderView: some View {
        ZStack {
            Circle()
                .fill(Color.accentColor.gradient)
            
            Text(initials)
                .font(.system(size: size * 0.4, weight: .semibold))
                .foregroundStyle(.white)
        }
    }
}

// Usage
AvatarView(imageURL: user.avatarURL, initials: "JD", size: 60)
```

---

## Badge View

```swift
struct BadgeView: View {
    let text: String
    let color: Color
    
    init(text: String, color: Color = .accentColor) {
        self.text = text
        self.color = color
    }
    
    var body: some View {
        Text(text)
            .font(.caption)
            .fontWeight(.semibold)
            .foregroundStyle(.white)
            .padding(.horizontal, 8)
            .padding(.vertical, 4)
            .background(color)
            .cornerRadius(8)
    }
}

// Usage
BadgeView(text: "New", color: .green)
BadgeView(text: "Premium", color: .orange)
```

---

## Custom Text Field

```swift
struct CustomTextField: View {
    let title: String
    let placeholder: String
    @Binding var text: String
    var isSecure: Bool = false
    var keyboardType: UIKeyboardType = .default
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(title)
                .font(.subheadline)
                .fontWeight(.semibold)
                .foregroundStyle(.secondary)
            
            Group {
                if isSecure {
                    SecureField(placeholder, text: $text)
                } else {
                    TextField(placeholder, text: $text)
                        .keyboardType(keyboardType)
                }
            }
            .textFieldStyle(.plain)
            .padding()
            .background(Color(.secondarySystemBackground))
            .cornerRadius(8)
        }
    }
}

// Usage
CustomTextField(
    title: "Email",
    placeholder: "Enter your email",
    text: $email,
    keyboardType: .emailAddress
)

CustomTextField(
    title: "Password",
    placeholder: "Enter password",
    text: $password,
    isSecure: true
)
```

---

## Section Header

```swift
struct SectionHeaderView: View {
    let title: String
    let actionTitle: String?
    let action: (() -> Void)?
    
    init(title: String, actionTitle: String? = nil, action: (() -> Void)? = nil) {
        self.title = title
        self.actionTitle = actionTitle
        self.action = action
    }
    
    var body: some View {
        HStack {
            Text(title)
                .font(.headline)
            
            Spacer()
            
            if let actionTitle, let action {
                Button(actionTitle, action: action)
                    .font(.subheadline)
            }
        }
        .padding(.horizontal)
        .padding(.vertical, 8)
    }
}

// Usage
ScrollView {
    SectionHeaderView(title: "Recent", actionTitle: "See All") {
        // Navigate to all items
    }
    // Content
}
```

---

## Best Practices

### 1. Make Components Reusable

```swift
// ✅ Good - Flexible with parameters
struct CustomButton: View {
    let title: String
    let icon: String?
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            HStack {
                if let icon {
                    Image(systemName: icon)
                }
                Text(title)
            }
        }
    }
}

// ❌ Bad - Too specific
struct LoginButton: View {
    // Hard-coded for one use case
}
```

### 2. Use ViewBuilder for Flexible Content

```swift
struct Container<Content: View>: View {
    @ViewBuilder let content: () -> Content
    
    var body: some View {
        content()
            .padding()
            .background(Color.gray.opacity(0.1))
    }
}
```

### 3. Provide Sensible Defaults

```swift
struct IconButton: View {
    let icon: String
    let size: CGFloat
    let color: Color
    
    init(
        icon: String,
        size: CGFloat = 24,
        color: Color = .accentColor
    ) {
        self.icon = icon
        self.size = size
        self.color = color
    }
}
```

### 4. Support Accessibility

```swift
Button(action: deleteItem) {
    Image(systemName: "trash")
}
.accessibilityLabel("Delete item")
.accessibilityHint("Removes this item from your list")
```

---

## Component Library Structure

Organize components by purpose:

```
Components/
├── Buttons/
│   ├── PrimaryButton.swift
│   └── SecondaryButton.swift
├── Cards/
│   └── CardView.swift
├── Forms/
│   └── CustomTextField.swift
├── Indicators/
│   ├── LoadingView.swift
│   └── BadgeView.swift
└── EmptyStates/
    ├── EmptyStateView.swift
    └── ErrorView.swift
```

---

## Key Takeaways

- ✅ Create reusable components with parameters
- ✅ Provide sensible defaults
- ✅ Use ViewBuilder for flexible content
- ✅ Support accessibility
- ✅ Keep styling consistent
- ✅ Document usage with examples
- ✅ Test components in isolation
