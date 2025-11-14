# iOS Best Practices

## Arquitectura
- ✅ Usa MVVM para apps SwiftUI
- ✅ Separa lógica de negocio de UI
- ✅ Implementa Repository Pattern
- ✅ Usa Dependency Injection
- ✅ Aplica SOLID principles

## Código Swift
```swift
// ✅ Prefiere let sobre var
let constant = "value"
var variable = "value"

// ✅ Usa guard para early returns
func process(value: String?) {
    guard let value = value else { return }
    // Usar value
}

// ✅ Evita force unwrapping
let text = optional ?? "default" // ✅
// let text = optional! // ❌

// ✅ Usa meaningful names
let userEmailAddress = "user@example.com" // ✅
let x = "user@example.com" // ❌
```

## SwiftUI
```swift
// ✅ Usa @State para estado local
@State private var isExpanded = false

// ✅ Usa @StateObject para ViewModels
@StateObject private var viewModel = ViewModel()

// ✅ Mantén vistas pequeñas
struct ContentView: View {
    var body: some View {
        VStack {
            HeaderView()
            BodyView()
            FooterView()
        }
    }
}

// ✅ Usa @MainActor en ViewModels
@MainActor
class ViewModel: ObservableObject {
    @Published var data: [Item] = []
}
```

## Concurrencia
```swift
// ✅ Usa async/await sobre callbacks
func fetchData() async throws -> Data {
    try await URLSession.shared.data(from: url).0
}

// ✅ Usa Actors para estado compartido
actor Cache {
    private var data: [String: Data] = [:]
    
    func get(_ key: String) -> Data? {
        data[key]
    }
}

// ✅ Usa Task para operaciones async
Task {
    let data = try await fetchData()
}
```

## Networking
```swift
// ✅ Maneja errores apropiadamente
func fetchUsers() async throws -> [User] {
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw NetworkError.invalidResponse
    }
    
    return try JSONDecoder().decode([User].self, from: data)
}

// ✅ Usa generic methods
func request<T: Decodable>(_ type: T.Type, from url: URL) async throws -> T {
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(T.self, from: data)
}
```

## Performance
```swift
// ✅ Lazy loading de imágenes
AsyncImage(url: imageURL)

// ✅ Usa LazyVStack para listas largas
ScrollView {
    LazyVStack {
        ForEach(items) { item in
            ItemView(item: item)
        }
    }
}

// ✅ Evita retain cycles
class ViewController {
    var closure: (() -> Void)?
    
    func setup() {
        closure = { [weak self] in
            self?.doSomething()
        }
    }
}
```

## Seguridad
```swift
// ✅ Usa Keychain para datos sensibles
import Security

func saveToKeychain(key: String, data: Data) {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key,
        kSecValueData as String: data
    ]
    SecItemAdd(query as CFDictionary, nil)
}

// ✅ Valida inputs del usuario
func validateEmail(_ email: String) -> Bool {
    email.contains("@") && email.contains(".")
}
```

## Testing
```swift
// ✅ Unit tests para lógica de negocio
class ViewModelTests: XCTestCase {
    func testFetchUsers() async throws {
        let viewModel = ViewModel()
        await viewModel.loadUsers()
        XCTAssertFalse(viewModel.users.isEmpty)
    }
}

// ✅ Usa mocks para dependencies
class MockRepository: UserRepository {
    func fetchUsers() async throws -> [User] {
        [User(id: "1", name: "Test")]
    }
}
```

## Code Organization
```swift
// ✅ Organiza código en extensions
extension UserViewModel {
    // MARK: - Actions
    func loadData() async { }
    
    // MARK: - Helpers
    private func validate() -> Bool { true }
}

// ✅ Usa MARK comments
class MyClass {
    // MARK: - Properties
    var name: String
    
    // MARK: - Lifecycle
    init() { }
    
    // MARK: - Methods
    func doSomething() { }
}
```

## Naming Conventions
```swift
// ✅ Classes, Structs: PascalCase
class UserViewModel { }
struct User { }

// ✅ Variables, Functions: camelCase
var userName = ""
func fetchData() { }

// ✅ Constants: camelCase
let maxRetries = 3
let apiBaseURL = "https://api.example.com"

// ✅ Protocols: descriptive, often ending in -able, -ing
protocol Drivable { }
protocol Networking { }
```

## Git & Version Control
```bash
# ✅ Commits descriptivos
git commit -m "Add user authentication feature"

# ✅ Usa branches para features
git checkout -b feature/user-profile

# ✅ Pull requests con descripción
```

## Resources
- 📚 [Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)
- 📚 [Apple HIG](https://developer.apple.com/design/human-interface-guidelines)
- 🎥 [WWDC Sessions](https://developer.apple.com/videos/)
- 📝 [Swift Style Guide](https://google.github.io/swift/)
