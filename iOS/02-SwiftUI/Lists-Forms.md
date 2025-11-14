# Lists & Forms en SwiftUI

## List Básica

```swift
struct ContentView: View {
    let items = ["Item 1", "Item 2", "Item 3"]
    
    var body: some View {
        List(items, id: \.self) { item in
            Text(item)
        }
    }
}

// Con modelos Identifiable
struct User: Identifiable {
    let id = UUID()
    let name: String
}

struct UserListView: View {
    let users = [
        User(name: "Ana"),
        User(name: "Luis")
    ]
    
    var body: some View {
        List(users) { user in
            Text(user.name)
        }
    }
}
```

## List con Secciones

```swift
struct GroupedListView: View {
    var body: some View {
        List {
            Section("Favorites") {
                Text("Item 1")
                Text("Item 2")
            }
            
            Section("Recent") {
                Text("Item 3")
                Text("Item 4")
            }
        }
        .listStyle(.grouped)
    }
}
```

## Swipe Actions

```swift
struct SwipeListView: View {
    @State private var items = ["Item 1", "Item 2", "Item 3"]
    
    var body: some View {
        List {
            ForEach(items, id: \.self) { item in
                Text(item)
                    .swipeActions {
                        Button(role: .destructive) {
                            items.removeAll { $0 == item }
                        } label: {
                            Label("Delete", systemImage: "trash")
                        }
                    }
            }
        }
    }
}
```

## Forms

```swift
struct SettingsView: View {
    @State private var name = ""
    @State private var isEnabled = false
    @State private var selectedOption = 0
    
    var body: some View {
        Form {
            Section("Profile") {
                TextField("Name", text: $name)
                Toggle("Notifications", isOn: $isEnabled)
            }
            
            Section("Options") {
                Picker("Select", selection: $selectedOption) {
                    Text("Option 1").tag(0)
                    Text("Option 2").tag(1)
                }
            }
            
            Section {
                Button("Save") {
                    // Save action
                }
            }
        }
    }
}
```

## Recursos

- 📚 [Lists](https://developer.apple.com/documentation/swiftui/list)
- 📚 [Forms](https://developer.apple.com/documentation/swiftui/form)
