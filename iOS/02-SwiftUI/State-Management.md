# State Management en SwiftUI - Guía Completa

## Tabla de Contenido
1. [@State](#state)
2. [@Binding](#binding)
3. [@StateObject y @ObservedObject](#stateobject-y-observedobject)
4. [@EnvironmentObject](#environmentobject)
5. [@Environment](#environment)
6. [@AppStorage y @SceneStorage](#appstorage-y-scenestorage)
7. [Observable Macro (iOS 17+)](#observable-macro-ios-17)
8. [Best Practices](#best-practices)
9. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
10. [Próximos Pasos](#próximos-pasos)
11. [Recursos](#recursos)

---

## @State

`@State` es una property wrapper para manejar estado local **dentro de una vista**. Cuando cambia, SwiftUI automáticamente redibuja la vista.

### Básico

```swift
struct CounterView: View {
    @State private var count = 0
    
    var body: some View {
        VStack {
            Text("Count: \(count)")
                .font(.largeTitle)
            
            Button("Increment") {
                count += 1
            }
            .buttonStyle(.borderedProminent)
        }
    }
}
```

### ¿Cuándo usar @State?

```swift
// ✅ Bueno - Estado local simple
@State private var isExpanded = false
@State private var selectedTab = 0
@State private var sliderValue = 0.5

// ❌ Malo - Datos complejos o compartidos
@State private var users: [User] = [] // Usar @StateObject en su lugar
@State private var networkManager = NetworkManager() // Usar @StateObject
```

### Tipos de Datos con @State

```swift
struct FormView: View {
    // Primitivos
    @State private var text = ""
    @State private var number = 0
    @State private var isOn = false
    
    // Colecciones simples
    @State private var items: [String] = []
    @State private var selectedItems: Set<String> = []
    
    // Structs simples
    @State private var date = Date()
    @State private var color = Color.blue
    
    var body: some View {
        Form {
            TextField("Text", text: $text)
            Stepper("Number: \(number)", value: $number)
            Toggle("Switch", isOn: $isOn)
            DatePicker("Date", selection: $date)
            ColorPicker("Color", selection: $color)
        }
    }
}
```

### Modificando @State

```swift
struct TodoView: View {
    @State private var todos: [String] = []
    @State private var newTodo = ""
    
    var body: some View {
        VStack {
            HStack {
                TextField("New todo", text: $newTodo)
                Button("Add") {
                    todos.append(newTodo)
                    newTodo = ""
                }
            }
            
            List {
                ForEach(todos, id: \.self) { todo in
                    Text(todo)
                }
                .onDelete { indexSet in
                    todos.remove(atOffsets: indexSet)
                }
            }
        }
    }
}
```

---

## @Binding

`@Binding` crea una **conexión de dos vías** entre la vista padre y la vista hija, permitiendo que ambas lean y modifiquen el mismo estado.

### Básico

```swift
// Vista Padre
struct ParentView: View {
    @State private var isOn = false
    
    var body: some View {
        VStack {
            Text("Switch is \(isOn ? "ON" : "OFF")")
            
            // Pasando binding con $
            ChildToggle(isOn: $isOn)
        }
    }
}

// Vista Hija
struct ChildToggle: View {
    @Binding var isOn: Bool
    
    var body: some View {
        Toggle("Switch", isOn: $isOn)
    }
}
```

### Bindings Derivados

```swift
struct SettingsView: View {
    @State private var settings = Settings()
    
    var body: some View {
        Form {
            // Binding a una propiedad específica
            Toggle("Notifications", isOn: $settings.notificationsEnabled)
            Toggle("Dark Mode", isOn: $settings.darkModeEnabled)
            
            // Binding computado
            NotificationSettings(
                isEnabled: Binding(
                    get: { settings.notificationsEnabled },
                    set: { settings.notificationsEnabled = $0 }
                )
            )
        }
    }
}

struct Settings {
    var notificationsEnabled = false
    var darkModeEnabled = false
}
```

### Custom Bindings

```swift
struct CustomBindingView: View {
    @State private var value = 0.0
    
    // Binding que valida el valor
    var validatedBinding: Binding<Double> {
        Binding(
            get: { value },
            set: { newValue in
                value = min(max(newValue, 0), 100) // Clamp entre 0-100
            }
        )
    }
    
    var body: some View {
        VStack {
            Slider(value: validatedBinding, in: 0...100)
            Text("Value: \(value, specifier: "%.1f")")
        }
    }
}
```

### Constant Bindings

```swift
struct PreviewView: View {
    var body: some View {
        // Binding constante para previews
        Toggle("Preview", isOn: .constant(true))
        TextField("Name", text: .constant("Preview Text"))
    }
}
```

---

## @StateObject y @ObservedObject

### @StateObject - Crear y poseer el objeto

```swift
class UserViewModel: ObservableObject {
    @Published var name = ""
    @Published var age = 0
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    func fetchUsers() async {
        isLoading = true
        errorMessage = nil
        
        do {
            // Simulate API call
            try await Task.sleep(nanoseconds: 1_000_000_000)
            users = [
                User(id: 1, name: "Ana"),
                User(id: 2, name: "Luis")
            ]
        } catch {
            errorMessage = error.localizedDescription
        }
        
        isLoading = false
    }
    
    func addUser(name: String) {
        let newUser = User(id: users.count + 1, name: name)
        users.append(newUser)
    }
}

struct User: Identifiable {
    let id: Int
    let name: String
}

struct UserListView: View {
    @StateObject private var viewModel = UserViewModel()
    
    var body: some View {
        NavigationStack {
            List(viewModel.users) { user in
                Text(user.name)
            }
            .navigationTitle("Users")
            .task {
                await viewModel.fetchUsers()
            }
            .overlay {
                if viewModel.isLoading {
                    ProgressView()
                }
            }
        }
    }
}
```

### @ObservedObject - Observar objeto existente

```swift
struct DetailView: View {
    @ObservedObject var viewModel: UserViewModel
    @State private var newName = ""
    
    var body: some View {
        Form {
            Section("Add User") {
                TextField("Name", text: $newName)
                Button("Add") {
                    viewModel.addUser(name: newName)
                    newName = ""
                }
            }
            
            Section("Users") {
                ForEach(viewModel.users) { user in
                    Text(user.name)
                }
            }
        }
    }
}
```

### ¿Cuándo usar cada uno?

```swift
// ✅ @StateObject - Vista CREA el objeto
struct ProfileView: View {
    @StateObject private var viewModel = ProfileViewModel()
    // El objeto persiste mientras la vista exista
}

// ✅ @ObservedObject - Vista RECIBE el objeto
struct ProfileDetailView: View {
    @ObservedObject var viewModel: ProfileViewModel
    // El objeto viene de otra vista
}

// ❌ Malo - Crear con @ObservedObject
struct WrongView: View {
    @ObservedObject var viewModel = ViewModel() // Se recrea en cada render!
}
```

---

## @EnvironmentObject

Compartir datos globalmente sin pasar por cada vista intermedia.

### Setup Global

```swift
class AppState: ObservableObject {
    @Published var isLoggedIn = false
    @Published var currentUser: User?
    @Published var theme: Theme = .light
    
    enum Theme {
        case light, dark
    }
    
    func login(user: User) {
        currentUser = user
        isLoggedIn = true
    }
    
    func logout() {
        currentUser = nil
        isLoggedIn = false
    }
}

@main
struct MyApp: App {
    @StateObject private var appState = AppState()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)
        }
    }
}
```

### Uso en Vistas

```swift
struct ContentView: View {
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        if appState.isLoggedIn {
            HomeView()
        } else {
            LoginView()
        }
    }
}

struct HomeView: View {
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        VStack {
            Text("Welcome, \(appState.currentUser?.name ?? "User")")
            
            Button("Logout") {
                appState.logout()
            }
        }
    }
}

struct LoginView: View {
    @EnvironmentObject var appState: AppState
    @State private var username = ""
    
    var body: some View {
        VStack {
            TextField("Username", text: $username)
            
            Button("Login") {
                let user = User(id: 1, name: username)
                appState.login(user: user)
            }
        }
    }
}
```

### Múltiples EnvironmentObjects

```swift
class SettingsManager: ObservableObject {
    @Published var fontSize: CGFloat = 16
    @Published var isDarkMode = false
}

class DataManager: ObservableObject {
    @Published var items: [Item] = []
}

@main
struct MyApp: App {
    @StateObject private var appState = AppState()
    @StateObject private var settings = SettingsManager()
    @StateObject private var dataManager = DataManager()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)
                .environmentObject(settings)
                .environmentObject(dataManager)
        }
    }
}

struct SomeView: View {
    @EnvironmentObject var appState: AppState
    @EnvironmentObject var settings: SettingsManager
    @EnvironmentObject var dataManager: DataManager
    
    var body: some View {
        // Usar todos los environment objects
    }
}
```

---

## @Environment

Acceder a valores del sistema proporcionados por SwiftUI.

### Valores Comunes

```swift
struct EnvironmentView: View {
    @Environment(\.colorScheme) var colorScheme
    @Environment(\.dismiss) var dismiss
    @Environment(\.scenePhase) var scenePhase
    @Environment(\.horizontalSizeClass) var sizeClass
    @Environment(\.openURL) var openURL
    
    var body: some View {
        VStack {
            Text("Theme: \(colorScheme == .dark ? "Dark" : "Light")")
            
            Button("Dismiss") {
                dismiss()
            }
            
            Button("Open URL") {
                openURL(URL(string: "https://apple.com")!)
            }
        }
        .onChange(of: scenePhase) { oldPhase, newPhase in
            if newPhase == .active {
                print("App is active")
            }
        }
    }
}
```

### Custom Environment Keys

```swift
// 1. Definir el key
struct UserIDKey: EnvironmentKey {
    static let defaultValue: String? = nil
}

// 2. Extender EnvironmentValues
extension EnvironmentValues {
    var userID: String? {
        get { self[UserIDKey.self] }
        set { self[UserIDKey.self] = newValue }
    }
}

// 3. Usar en vistas
struct ParentView: View {
    var body: some View {
        ChildView()
            .environment(\.userID, "user123")
    }
}

struct ChildView: View {
    @Environment(\.userID) var userID
    
    var body: some View {
        Text("User ID: \(userID ?? "None")")
    }
}
```

---

## @AppStorage y @SceneStorage

### @AppStorage - UserDefaults

```swift
struct SettingsView: View {
    @AppStorage("username") private var username = ""
    @AppStorage("fontSize") private var fontSize = 16.0
    @AppStorage("isDarkMode") private var isDarkMode = false
    @AppStorage("selectedTab") private var selectedTab = 0
    
    var body: some View {
        Form {
            TextField("Username", text: $username)
            
            Slider(value: $fontSize, in: 12...24) {
                Text("Font Size: \(Int(fontSize))")
            }
            
            Toggle("Dark Mode", isOn: $isDarkMode)
        }
    }
}
```

### @SceneStorage - State Restoration

```swift
struct ContentView: View {
    @SceneStorage("selectedTab") private var selectedTab = 0
    @SceneStorage("searchText") private var searchText = ""
    
    var body: some View {
        TabView(selection: $selectedTab) {
            Text("Home")
                .tabItem { Label("Home", systemImage: "house") }
                .tag(0)
            
            Text("Search")
                .tabItem { Label("Search", systemImage: "magnifyingglass") }
                .tag(1)
        }
    }
}
```

---

## Observable Macro (iOS 17+)

Nueva forma de crear objetos observables con menos boilerplate.

```swift
import Observation

@Observable
class UserViewModel {
    var name = ""
    var age = 0
    var users: [User] = []
    var isLoading = false
    
    func fetchUsers() async {
        isLoading = true
        // Fetch logic
        isLoading = false
    }
}

struct UserView: View {
    @State private var viewModel = UserViewModel()
    
    var body: some View {
        List(viewModel.users) { user in
            Text(user.name)
        }
        .task {
            await viewModel.fetchUsers()
        }
    }
}
```

---

## Best Practices

### 1. Ownership Claro

```swift
// ✅ Bueno
struct ParentView: View {
    @StateObject private var viewModel = ViewModel() // Parent owns
    
    var body: some View {
        ChildView(viewModel: viewModel) // Child observes
    }
}

struct ChildView: View {
    @ObservedObject var viewModel: ViewModel
}
```

### 2. Estado Mínimo

```swift
// ❌ Malo - Estado redundante
@State private var fullName = ""
@State private var firstName = ""
@State private var lastName = ""

// ✅ Bueno - Computed property
@State private var firstName = ""
@State private var lastName = ""
var fullName: String {
    "\(firstName) \(lastName)"
}
```

### 3. Private cuando sea posible

```swift
struct MyView: View {
    @State private var count = 0 // ✅ Private
    @StateObject private var viewModel = ViewModel() // ✅ Private
}
```

---

## Checklist de Aprendizaje

### Básico
- [ ] Usar @State para estado local simple
- [ ] Crear @Binding entre padre e hijo
- [ ] Entender diferencia @StateObject vs @ObservedObject
- [ ] Usar @Environment para valores del sistema

### Intermedio
- [ ] Compartir datos con @EnvironmentObject
- [ ] Crear custom Environment keys
- [ ] Usar @AppStorage para persistencia
- [ ] Implementar ViewModels con ObservableObject

### Avanzado
- [ ] Optimizar re-renders
- [ ] Usar @Observable (iOS 17+)
- [ ] Custom Bindings complejos
- [ ] State management patterns

---

## Próximos Pasos

### 1. Patterns Avanzados
- Unidirectional Data Flow
- Redux-like architecture
- TCA (The Composable Architecture)

### 2. Performance
- Minimizar re-renders
- Usar @ObservedObject juiciosamente
- Profile con Instruments

---

## Recursos

### Documentación
- 📚 [State and Data Flow](https://developer.apple.com/documentation/swiftui/state-and-data-flow)
- 📖 [Managing Model Data](https://developer.apple.com/documentation/swiftui/managing-model-data-in-your-app)

### Videos
- 🎥 [SwiftUI State Management](https://www.youtube.com/watch?v=3krC2c56ceQ)
- 🎥 [Data Flow in SwiftUI](https://developer.apple.com/videos/play/wwdc2019/226/)

### Artículos
- 📝 [Hacking with Swift - Property Wrappers](https://www.hackingwithswift.com/quick-start/swiftui/all-swiftui-property-wrappers-explained)
- 📝 [Swift by Sundell - State Management](https://www.swiftbysundell.com/articles/swiftui-state-management-guide/)

---

**Anterior:** [Swift Basics](../01-Fundamentos-Swift/Swift-Basics.md)  
**Próximo:** [Navigation](Navigation.md)
