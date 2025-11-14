# Navigation en SwiftUI - Guía Completa

## Tabla de Contenido
1. [NavigationStack (iOS 16+)](#navigationstack-ios-16)
2. [NavigationPath](#navigationpath)
3. [NavigationLink](#navigationlink)
4. [Programmatic Navigation](#programmatic-navigation)
5. [Sheet Presentation](#sheet-presentation)
6. [FullScreenCover](#fullscreencover)
7. [Alert y Confirmation Dialog](#alert-y-confirmation-dialog)
8. [NavigationSplitView](#navigationsplitview)
9. [Deep Linking](#deep-linking)
10. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
11. [Próximos Pasos](#próximos-pasos)
12. [Recursos](#recursos)

---

## NavigationStack (iOS 16+)

NavigationStack reemplaza NavigationView con un modelo más robusto y type-safe.

### Básico

```swift
struct ContentView: View {
    var body: some View {
        NavigationStack {
            List(1...20, id: \.self) { number in
                NavigationLink("Item \(number)", value: number)
            }
            .navigationDestination(for: Int.self) { number in
                DetailView(number: number)
            }
            .navigationTitle("List")
            .navigationBarTitleDisplayMode(.large)
        }
    }
}

struct DetailView: View {
    let number: Int
    
    var body: some View {
        Text("Detail \(number)")
            .font(.largeTitle)
            .navigationTitle("Detail")
            .navigationBarTitleDisplayMode(.inline)
    }
}
```

### Múltiples Tipos de Destinos

```swift
enum Route: Hashable {
    case user(User)
    case settings
    case about
}

struct HomeView: View {
    var body: some View {
        NavigationStack {
            List {
                NavigationLink("User Profile", value: Route.user(User(id: 1, name: "Ana")))
                NavigationLink("Settings", value: Route.settings)
                NavigationLink("About", value: Route.about)
            }
            .navigationDestination(for: Route.self) { route in
                switch route {
                case .user(let user):
                    UserDetailView(user: user)
                case .settings:
                    SettingsView()
                case .about:
                    AboutView()
                }
            }
            .navigationTitle("Home")
        }
    }
}
```

### Navigation con Datos Complejos

```swift
struct Product: Hashable, Identifiable {
    let id: Int
    let name: String
    let price: Double
}

struct ProductListView: View {
    let products = [
        Product(id: 1, name: "iPhone", price: 999),
        Product(id: 2, name: "iPad", price: 799),
        Product(id: 3, name: "MacBook", price: 1299)
    ]
    
    var body: some View {
        NavigationStack {
            List(products) { product in
                NavigationLink(value: product) {
                    HStack {
                        Text(product.name)
                        Spacer()
                        Text("$\(Int(product.price))")
                            .foregroundColor(.secondary)
                    }
                }
            }
            .navigationDestination(for: Product.self) { product in
                ProductDetailView(product: product)
            }
            .navigationTitle("Products")
        }
    }
}

struct ProductDetailView: View {
    let product: Product
    
    var body: some View {
        VStack(spacing: 20) {
            Text(product.name)
                .font(.title)
            
            Text("$\(Int(product.price))")
                .font(.title2)
                .foregroundColor(.green)
            
            Button("Buy Now") {
                // Purchase logic
            }
            .buttonStyle(.borderedProminent)
        }
        .navigationTitle(product.name)
    }
}
```

---

## NavigationPath

NavigationPath permite control programático sobre la navegación.

### Básico

```swift
@Observable
class NavigationManager {
    var path = NavigationPath()
    
    func navigateToDetail(_ number: Int) {
        path.append(number)
    }
    
    func navigateBack() {
        if !path.isEmpty {
            path.removeLast()
        }
    }
    
    func popToRoot() {
        path = NavigationPath()
    }
}

struct AppView: View {
    @State private var manager = NavigationManager()
    
    var body: some View {
        NavigationStack(path: $manager.path) {
            VStack {
                Button("Navigate to 1") {
                    manager.navigateToDetail(1)
                }
                
                Button("Navigate to 2") {
                    manager.navigateToDetail(2)
                }
                
                Button("Pop to Root") {
                    manager.popToRoot()
                }
            }
            .navigationDestination(for: Int.self) { number in
                DetailWithNavigation(number: number, manager: manager)
            }
            .navigationTitle("Home")
        }
    }
}

struct DetailWithNavigation: View {
    let number: Int
    let manager: NavigationManager
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Detail \(number)")
                .font(.title)
            
            Button("Go Deeper (\(number + 1))") {
                manager.navigateToDetail(number + 1)
            }
            
            Button("Back") {
                manager.navigateBack()
            }
            
            Button("Pop to Root") {
                manager.popToRoot()
            }
        }
        .navigationTitle("Detail \(number)")
    }
}
```

### NavigationPath con Tipos Múltiples

```swift
@Observable
class AppNavigator {
    var path = NavigationPath()
    
    func navigate(to destination: any Hashable) {
        path.append(destination)
    }
    
    func popToRoot() {
        path.removeLast(path.count)
    }
}

struct MultiTypeNav: View {
    @State private var navigator = AppNavigator()
    
    var body: some View {
        NavigationStack(path: $navigator.path) {
            List {
                Button("Show User") {
                    navigator.navigate(to: User(id: 1, name: "Ana"))
                }
                
                Button("Show Settings") {
                    navigator.navigate(to: "settings")
                }
                
                Button("Show Number") {
                    navigator.navigate(to: 42)
                }
            }
            .navigationDestination(for: User.self) { user in
                UserView(user: user)
            }
            .navigationDestination(for: String.self) { route in
                if route == "settings" {
                    SettingsView()
                }
            }
            .navigationDestination(for: Int.self) { number in
                NumberView(number: number)
            }
        }
    }
}
```

---

## NavigationLink

### NavigationLink con Value

```swift
struct UserListView: View {
    let users = [
        User(id: 1, name: "Ana"),
        User(id: 2, name: "Luis"),
        User(id: 3, name: "María")
    ]
    
    var body: some View {
        NavigationStack {
            List(users) { user in
                NavigationLink(value: user) {
                    UserRow(user: user)
                }
            }
            .navigationDestination(for: User.self) { user in
                UserDetailView(user: user)
            }
        }
    }
}

struct UserRow: View {
    let user: User
    
    var body: some View {
        HStack {
            Image(systemName: "person.circle.fill")
                .font(.title)
                .foregroundColor(.blue)
            
            VStack(alignment: .leading) {
                Text(user.name)
                    .font(.headline)
                Text("ID: \(user.id)")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
    }
}
```

### NavigationLink con Destination

```swift
struct OldStyleNav: View {
    var body: some View {
        NavigationStack {
            List {
                NavigationLink {
                    Text("Destination")
                } label: {
                    Text("Go to Destination")
                }
            }
        }
    }
}
```

---

## Programmatic Navigation

### Con Bool

```swift
struct ProgrammaticNav: View {
    @State private var showDetail = false
    
    var body: some View {
        NavigationStack {
            VStack {
                Button("Show Detail") {
                    showDetail = true
                }
                
                NavigationLink(
                    destination: DetailView(),
                    isActive: $showDetail
                ) {
                    EmptyView()
                }
                .hidden()
            }
            .navigationTitle("Home")
        }
    }
}
```

### Con Optional

```swift
struct SelectionNav: View {
    @State private var selectedUser: User?
    
    var body: some View {
        NavigationStack {
            List(users) { user in
                Button(user.name) {
                    selectedUser = user
                }
            }
            .navigationDestination(item: $selectedUser) { user in
                UserDetailView(user: user)
            }
        }
    }
}
```

---

## Sheet Presentation

### Sheet Básico

```swift
struct SheetExample: View {
    @State private var showSheet = false
    
    var body: some View {
        Button("Show Sheet") {
            showSheet = true
        }
        .sheet(isPresented: $showSheet) {
            SheetContentView()
        }
    }
}

struct SheetContentView: View {
    @Environment(\.dismiss) var dismiss
    
    var body: some View {
        NavigationStack {
            VStack {
                Text("Sheet Content")
                    .font(.title)
                
                Button("Dismiss") {
                    dismiss()
                }
            }
            .navigationTitle("Sheet")
            .navigationBarTitleDisplayMode(.inline)
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

### Sheet con Item

```swift
struct SheetWithItem: View {
    @State private var selectedUser: User?
    
    var body: some View {
        List(users) { user in
            Button(user.name) {
                selectedUser = user
            }
        }
        .sheet(item: $selectedUser) { user in
            UserSheetView(user: user)
        }
    }
}
```

### Presentation Detents (iOS 16+)

```swift
struct DetentsExample: View {
    @State private var showSheet = false
    
    var body: some View {
        Button("Show Sheet") {
            showSheet = true
        }
        .sheet(isPresented: $showSheet) {
            Text("Resizable Sheet")
                .presentationDetents([.medium, .large])
                .presentationDragIndicator(.visible)
        }
    }
}
```

### Multiple Sheets

```swift
struct MultipleSheets: View {
    @State private var showSettings = false
    @State private var showProfile = false
    
    var body: some View {
        VStack {
            Button("Settings") {
                showSettings = true
            }
            
            Button("Profile") {
                showProfile = true
            }
        }
        .sheet(isPresented: $showSettings) {
            SettingsView()
        }
        .sheet(isPresented: $showProfile) {
            ProfileView()
        }
    }
}
```

---

## FullScreenCover

```swift
struct FullScreenExample: View {
    @State private var showFullScreen = false
    
    var body: some View {
        Button("Show Full Screen") {
            showFullScreen = true
        }
        .fullScreenCover(isPresented: $showFullScreen) {
            FullScreenContentView()
        }
    }
}

struct FullScreenContentView: View {
    @Environment(\.dismiss) var dismiss
    
    var body: some View {
        ZStack {
            Color.blue.ignoresSafeArea()
            
            VStack {
                Text("Full Screen Cover")
                    .font(.largeTitle)
                    .foregroundColor(.white)
                
                Button("Dismiss") {
                    dismiss()
                }
                .buttonStyle(.bordered)
                .tint(.white)
            }
        }
    }
}
```

---

## Alert y Confirmation Dialog

### Alert

```swift
struct AlertExample: View {
    @State private var showAlert = false
    @State private var message = ""
    
    var body: some View {
        Button("Show Alert") {
            showAlert = true
        }
        .alert("Title", isPresented: $showAlert) {
            Button("OK") { }
            Button("Cancel", role: .cancel) { }
        } message: {
            Text("This is the message")
        }
    }
}
```

### Confirmation Dialog

```swift
struct ConfirmationExample: View {
    @State private var showDialog = false
    
    var body: some View {
        Button("Show Options") {
            showDialog = true
        }
        .confirmationDialog("Choose an option", isPresented: $showDialog) {
            Button("Option 1") { }
            Button("Option 2") { }
            Button("Delete", role: .destructive) { }
            Button("Cancel", role: .cancel) { }
        }
    }
}
```

---

## NavigationSplitView

Para iPad y layouts más grandes.

```swift
struct SplitViewExample: View {
    @State private var selectedCategory: Category?
    @State private var selectedItem: Item?
    
    var body: some View {
        NavigationSplitView {
            // Sidebar
            List(categories, selection: $selectedCategory) { category in
                Text(category.name)
            }
            .navigationTitle("Categories")
        } content: {
            // Content (optional middle column)
            if let category = selectedCategory {
                List(category.items, selection: $selectedItem) { item in
                    Text(item.name)
                }
                .navigationTitle(category.name)
            }
        } detail: {
            // Detail
            if let item = selectedItem {
                ItemDetailView(item: item)
            } else {
                Text("Select an item")
            }
        }
    }
}
```

---

## Deep Linking

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .onOpenURL { url in
                    handleDeepLink(url)
                }
        }
    }
    
    func handleDeepLink(_ url: URL) {
        // myapp://user/123
        guard url.scheme == "myapp" else { return }
        
        let components = url.pathComponents
        if components.count == 3, components[1] == "user" {
            let userID = components[2]
            // Navigate to user
        }
    }
}
```

---

## Checklist de Aprendizaje

### Básico
- [ ] Usar NavigationStack básico
- [ ] Crear NavigationLinks
- [ ] Presentar sheets
- [ ] Usar @Environment(\.dismiss)

### Intermedio
- [ ] Usar NavigationPath para control programático
- [ ] Manejar múltiples tipos de destinos
- [ ] Usar presentation detents
- [ ] Implementar NavigationSplitView

### Avanzado
- [ ] Deep linking
- [ ] Custom navigation transitions
- [ ] Navigation state restoration
- [ ] Complex navigation flows

---

## Próximos Pasos

### 1. Patterns Avanzados
- Coordinator Pattern
- Router Pattern
- Navigation State Management

### 2. Best Practices
- Type-safe navigation
- Navigation testing
- Performance optimization

---

## Recursos

### Documentación
- 📚 [NavigationStack](https://developer.apple.com/documentation/swiftui/navigationstack)
- 📖 [NavigationPath](https://developer.apple.com/documentation/swiftui/navigationpath)
- 📘 [NavigationSplitView](https://developer.apple.com/documentation/swiftui/navigationsplitview)

### Videos
- 🎥 [The SwiftUI cookbook for navigation](https://developer.apple.com/videos/play/wwdc2022/10054/)
- 🎥 [Navigation in SwiftUI](https://www.youtube.com/watch?v=4GjXq2Sr55Q)

### Artículos
- 📝 [Hacking with Swift - Navigation](https://www.hackingwithswift.com/quick-start/swiftui/how-to-use-navigationstack-to-navigate-programmatically)
- 📝 [Swift by Sundell - Navigation](https://www.swiftbysundell.com/articles/swiftui-navigation-patterns/)

---

**Anterior:** [State Management](State-Management.md)  
**Próximo:** [Views & Modifiers](Views-Modifiers.md)
