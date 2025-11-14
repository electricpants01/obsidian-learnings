# State Management en SwiftUI

## Tabla de Contenido
1. [@State](#state)
2. [@Binding](#binding)
3. [@StateObject y @ObservedObject](#stateobject-y-observedobject)
4. [@EnvironmentObject](#environmentobject)
5. [@Environment](#environment)
6. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
7. [Próximos Pasos](#próximos-pasos)
8. [Recursos](#recursos)

---

## @State

State local para una vista. Cuando cambia, SwiftUI redibuja la vista.

```swift
struct CounterView: View {
    @State private var count = 0
    
    var body: some View {
        VStack {
            Text("Count: \(count)")
            
            Button("Increment") {
                count += 1
            }
        }
    }
}

// Ejemplo: Toggle
struct ToggleView: View {
    @State private var isOn = false
    
    var body: some View {
        Toggle("Enable", isOn: $isOn)
    }
}

// Ejemplo: TextField
struct FormView: View {
    @State private var name = ""
    @State private var email = ""
    
    var body: some View {
        Form {
            TextField("Name", text: $name)
            TextField("Email", text: $email)
        }
    }
}
```

---

## @Binding

Permite compartir estado entre vistas padre e hija.

```swift
struct ParentView: View {
    @State private var isPresented = false
    
    var body: some View {
        VStack {
            Button("Show Sheet") {
                isPresented = true
            }
            .sheet(isPresented: $isPresented) {
                ChildView(isPresented: $isPresented)
            }
        }
    }
}

struct ChildView: View {
    @Binding var isPresented: Bool
    
    var body: some View {
        Button("Dismiss") {
            isPresented = false
        }
    }
}

// Ejemplo: Custom Toggle
struct CustomToggle: View {
    @Binding var isOn: Bool
    
    var body: some View {
        Button(isOn ? "ON" : "OFF") {
            isOn.toggle()
        }
    }
}
```

---

## @StateObject y @ObservedObject

### @StateObject - Crear y poseer el objeto

```swift
class ViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    
    func loadUsers() async {
        isLoading = true
        // Fetch users
        isLoading = false
    }
}

struct UserListView: View {
    @StateObject private var viewModel = ViewModel()
    
    var body: some View {
        List(viewModel.users) { user in
            Text(user.name)
        }
        .task {
            await viewModel.loadUsers()
        }
    }
}
```

### @ObservedObject - Observar objeto existente

```swift
struct DetailView: View {
    @ObservedObject var viewModel: ViewModel
    
    var body: some View {
        // View implementation
    }
}
```

---

## @EnvironmentObject

Compartir datos globalmente sin pasar por cada vista.

```swift
class AppState: ObservableObject {
    @Published var isLoggedIn = false
    @Published var user: User?
    
    func login() {
        isLoggedIn = true
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
```

---

## @Environment

Acceder a valores del sistema.

```swift
struct MyView: View {
    @Environment(\.colorScheme) var colorScheme
    @Environment(\.dismiss) var dismiss
    
    var body: some View {
        VStack {
            Text(colorScheme == .dark ? "Dark Mode" : "Light Mode")
            
            Button("Dismiss") {
                dismiss()
            }
        }
    }
}
```

---

## Checklist de Aprendizaje

### Básico
- [ ] Usar @State para estado local
- [ ] Crear @Binding entre vistas
- [ ] Implementar @StateObject en ViewModels
- [ ] Usar @Environment para sistema

### Intermedio
- [ ] Compartir datos con @EnvironmentObject
- [ ] Entender diferencia @StateObject vs @ObservedObject
- [ ] Implementar MVVM pattern

### Avanzado
- [ ] Custom Environment Keys
- [ ] Performance optimization
- [ ] Combine con @Published

---

## Próximos Pasos

1. **MVVM con SwiftUI**
2. **Combine Framework**
3. **Navigation con State**

---

## Recursos

- 📚 [State and Data Flow](https://developer.apple.com/documentation/swiftui/state-and-data-flow)
- 🎥 [SwiftUI State Management](https://www.youtube.com/watch?v=3krC2c56ceQ)
- 📝 [Hacking with Swift - State](https://www.hackingwithswift.com/quick-start/swiftui/all-swiftui-property-wrappers-explained)
