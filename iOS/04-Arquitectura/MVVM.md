# MVVM en iOS/SwiftUI

## Tabla de Contenido
1. [¿Qué es MVVM?](#qué-es-mvvm)
2. [View](#view)
3. [ViewModel](#viewmodel)
4. [Model](#model)
5. [Casos de Uso Reales](#casos-de-uso-reales)
6. [Checklist](#checklist-de-aprendizaje)
7. [Recursos](#recursos)

---

## ¿Qué es MVVM?

**Model-View-ViewModel** separa la lógica de negocio de la UI.

```
View ←→ ViewModel ←→ Model
 ↑         ↑           ↑
 UI    Lógica      Datos
```

---

## View

```swift
struct UserListView: View {
    @StateObject private var viewModel = UserListViewModel()
    
    var body: some View {
        NavigationStack {
            List(viewModel.users) { user in
                UserRow(user: user)
            }
            .navigationTitle("Users")
            .task {
                await viewModel.loadUsers()
            }
        }
    }
}
```

---

## ViewModel

```swift
@MainActor
class UserListViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private let repository: UserRepository
    
    init(repository: UserRepository = UserRepository()) {
        self.repository = repository
    }
    
    func loadUsers() async {
        isLoading = true
        errorMessage = nil
        
        do {
            users = try await repository.fetchUsers()
        } catch {
            errorMessage = error.localizedDescription
        }
        
        isLoading = false
    }
    
    func deleteUser(_ user: User) async {
        do {
            try await repository.deleteUser(user)
            users.removeAll { $0.id == user.id }
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}
```

---

## Model

```swift
struct User: Identifiable, Codable {
    let id: Int
    let name: String
    let email: String
}

class UserRepository {
    func fetchUsers() async throws -> [User] {
        let url = URL(string: "https://api.example.com/users")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode([User].self, from: data)
    }
    
    func deleteUser(_ user: User) async throws {
        // Delete implementation
    }
}
```

---

## Casos de Uso Reales

### Login Flow

```swift
@MainActor
class LoginViewModel: ObservableObject {
    @Published var email = ""
    @Published var password = ""
    @Published var isLoading = false
    @Published var errorMessage: String?
    @Published var isLoggedIn = false
    
    private let authService: AuthService
    
    init(authService: AuthService = AuthService()) {
        self.authService = authService
    }
    
    func login() async {
        isLoading = true
        errorMessage = nil
        
        do {
            try await authService.login(email: email, password: password)
            isLoggedIn = true
        } catch {
            errorMessage = error.localizedDescription
        }
        
        isLoading = false
    }
    
    var isValidForm: Bool {
        !email.isEmpty && password.count >= 6
    }
}

struct LoginView: View {
    @StateObject private var viewModel = LoginViewModel()
    
    var body: some View {
        VStack {
            TextField("Email", text: $viewModel.email)
            SecureField("Password", text: $viewModel.password)
            
            if let error = viewModel.errorMessage {
                Text(error).foregroundColor(.red)
            }
            
            Button("Login") {
                Task {
                    await viewModel.login()
                }
            }
            .disabled(!viewModel.isValidForm || viewModel.isLoading)
        }
    }
}
```

---

## Checklist de Aprendizaje

### Básico
- [ ] Separar View, ViewModel, Model
- [ ] Usar @Published en ViewModel
- [ ] Inyectar dependencies

### Intermedio
- [ ] Testing de ViewModels
- [ ] Error handling robusto
- [ ] Navigation con Coordinators

### Avanzado
- [ ] Dependency Injection Container
- [ ] Repository Pattern
- [ ] Use Cases

---

## Recursos

- 📚 [MVVM Pattern](https://developer.apple.com/documentation/swiftui/model-data)
- 🎥 [MVVM in SwiftUI](https://www.youtube.com/watch?v=4GjXq2Sr55Q)
- 📝 [Swift by Sundell - MVVM](https://www.swiftbysundell.com/articles/different-flavors-of-dependency-injection-in-swift/)
