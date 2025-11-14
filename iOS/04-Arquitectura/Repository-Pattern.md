# Repository Pattern en iOS - Guía Completa

## Tabla de Contenido
1. [¿Qué es Repository Pattern?](#qué-es-repository-pattern)
2. [Ventajas del Patrón](#ventajas-del-patrón)
3. [Repository Protocol](#repository-protocol)
4. [Data Sources](#data-sources)
5. [Implementation](#implementation)
6. [Caching Strategies](#caching-strategies)
7. [Error Handling](#error-handling)
8. [Testing](#testing)
9. [Casos de Uso Reales](#casos-de-uso-reales)
10. [Best Practices](#best-practices)
11. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
12. [Próximos Pasos](#próximos-pasos)
13. [Recursos](#recursos)

---

## ¿Qué es Repository Pattern?

El **Repository Pattern** actúa como una capa de abstracción entre la lógica de negocio y las fuentes de datos, proporcionando una interfaz uniforme para acceder a datos sin importar su origen.

### Concepto

```
┌────────────────────────────────────────┐
│        Domain/Business Logic           │
│                                        │
│    ↓ Usa interface (Protocol)         │
└────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────┐
│         Repository Protocol            │
│  (Define qué operaciones están         │
│   disponibles)                         │
└────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────┐
│      Repository Implementation         │
│  (Decide cómo obtener los datos)      │
│                                        │
│   ┌────────────┐    ┌──────────────┐  │
│   │ Remote DS  │    │  Local DS    │  │
│   │  (API)     │    │  (Database)  │  │
│   └────────────┘    └──────────────┘  │
└────────────────────────────────────────┘
```

### Principios

```swift
// ✅ Principios del Repository Pattern

// 1. Single Source of Truth
// - Un solo punto para acceder a datos
// - Coordina entre múltiples data sources

// 2. Abstraction
// - Domain no conoce el origen de datos
// - Fácil cambiar implementación

// 3. Separation of Concerns
// - Repository maneja lógica de datos
// - Domain maneja lógica de negocio

// 4. Testability
// - Fácil crear mocks
// - Tests sin dependencias externas
```

---

## Ventajas del Patrón

### 1. Abstracción de Data Sources

```swift
// Domain Layer no sabe de dónde vienen los datos
protocol UserRepository {
    func fetchUsers() async throws -> [User]
}

// Puede venir de API
class APIUserRepository: UserRepository { }

// Puede venir de Base de Datos
class DatabaseUserRepository: UserRepository { }

// Puede venir de Mock para tests
class MockUserRepository: UserRepository { }
```

### 2. Centralización de Lógica de Datos

```swift
class UserRepositoryImpl: UserRepository {
    func fetchUsers() async throws -> [User] {
        // Toda la lógica de caching,
        // sincronización, validación, etc.
        // está en un solo lugar
    }
}
```

### 3. Facilita Testing

```swift
// Mock simple para tests
class MockUserRepository: UserRepository {
    var usersToReturn: [User] = []
    
    func fetchUsers() async throws -> [User] {
        usersToReturn
    }
}
```

---

## Repository Protocol

### Basic Protocol

```swift
protocol UserRepository {
    // READ
    func fetchUsers() async throws -> [User]
    func getUser(id: String) async throws -> User
    func searchUsers(query: String) async throws -> [User]
    
    // CREATE
    func createUser(_ user: User) async throws -> User
    
    // UPDATE
    func updateUser(_ user: User) async throws -> User
    
    // DELETE
    func deleteUser(id: String) async throws
}
```

### Generic Repository

```swift
protocol Repository {
    associatedtype Entity
    
    func getAll() async throws -> [Entity]
    func get(id: String) async throws -> Entity
    func create(_ entity: Entity) async throws -> Entity
    func update(_ entity: Entity) async throws -> Entity
    func delete(id: String) async throws
}

// Implementación específica
class UserRepositoryImpl: Repository {
    typealias Entity = User
    
    func getAll() async throws -> [User] {
        // Implementation
    }
    
    func get(id: String) async throws -> User {
        // Implementation
    }
    
    func create(_ entity: User) async throws -> User {
        // Implementation
    }
    
    func update(_ entity: User) async throws -> User {
        // Implementation
    }
    
    func delete(id: String) async throws {
        // Implementation
    }
}
```

### Protocol con Filtros

```swift
protocol ProductRepository {
    func fetchProducts() async throws -> [Product]
    func fetchProducts(category: Category) async throws -> [Product]
    func fetchProducts(priceRange: ClosedRange<Double>) async throws -> [Product]
    func searchProducts(query: String) async throws -> [Product]
    func getProduct(id: String) async throws -> Product
    func saveProduct(_ product: Product) async throws
}
```

---

## Data Sources

### Remote Data Source

```swift
protocol RemoteDataSource {
    func fetch<T: Decodable>(_ endpoint: String) async throws -> T
    func post<T: Encodable, R: Decodable>(_ endpoint: String, body: T) async throws -> R
    func put<T: Encodable, R: Decodable>(_ endpoint: String, body: T) async throws -> R
    func delete(_ endpoint: String) async throws
}

class APIDataSource: RemoteDataSource {
    private let baseURL = "https://api.example.com"
    
    func fetch<T: Decodable>(_ endpoint: String) async throws -> T {
        let url = URL(string: "\(baseURL)/\(endpoint)")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode(T.self, from: data)
    }
    
    func post<T: Encodable, R: Decodable>(_ endpoint: String, body: T) async throws -> R {
        let url = URL(string: "\(baseURL)/\(endpoint)")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.httpBody = try JSONEncoder().encode(body)
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        let (data, _) = try await URLSession.shared.data(for: request)
        return try JSONDecoder().decode(R.self, from: data)
    }
    
    func put<T: Encodable, R: Decodable>(_ endpoint: String, body: T) async throws -> R {
        let url = URL(string: "\(baseURL)/\(endpoint)")!
        var request = URLRequest(url: url)
        request.httpMethod = "PUT"
        request.httpBody = try JSONEncoder().encode(body)
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        let (data, _) = try await URLSession.shared.data(for: request)
        return try JSONDecoder().decode(R.self, from: data)
    }
    
    func delete(_ endpoint: String) async throws {
        let url = URL(string: "\(baseURL)/\(endpoint)")!
        var request = URLRequest(url: url)
        request.httpMethod = "DELETE"
        
        _ = try await URLSession.shared.data(for: request)
    }
}
```

### Local Data Source

```swift
protocol LocalDataSource {
    func save<T: Encodable>(_ items: [T], key: String) async throws
    func fetch<T: Decodable>(key: String) async throws -> [T]
    func delete(key: String) async throws
}

class UserDefaultsDataSource: LocalDataSource {
    private let userDefaults = UserDefaults.standard
    
    func save<T: Encodable>(_ items: [T], key: String) async throws {
        let data = try JSONEncoder().encode(items)
        userDefaults.set(data, forKey: key)
    }
    
    func fetch<T: Decodable>(key: String) async throws -> [T] {
        guard let data = userDefaults.data(forKey: key) else {
            throw DataSourceError.notFound
        }
        return try JSONDecoder().decode([T].self, from: data)
    }
    
    func delete(key: String) async throws {
        userDefaults.removeObject(forKey: key)
    }
}

class CoreDataSource: LocalDataSource {
    private let context: NSManagedObjectContext
    
    init(context: NSManagedObjectContext) {
        self.context = context
    }
    
    func save<T: Encodable>(_ items: [T], key: String) async throws {
        // Core Data implementation
    }
    
    func fetch<T: Decodable>(key: String) async throws -> [T] {
        // Core Data implementation
    }
    
    func delete(key: String) async throws {
        // Core Data implementation
    }
}

enum DataSourceError: Error {
    case notFound
    case encodingFailed
    case decodingFailed
}
```

---

## Implementation

### Basic Repository Implementation

```swift
class UserRepositoryImpl: UserRepository {
    private let remoteDataSource: RemoteDataSource
    private let localDataSource: LocalDataSource
    
    init(
        remoteDataSource: RemoteDataSource,
        localDataSource: LocalDataSource
    ) {
        self.remoteDataSource = remoteDataSource
        self.localDataSource = localDataSource
    }
    
    func fetchUsers() async throws -> [User] {
        do {
            // Try remote first
            let users: [User] = try await remoteDataSource.fetch("users")
            
            // Cache locally
            try await localDataSource.save(users, key: "users")
            
            return users
        } catch {
            // Fallback to local
            return try await localDataSource.fetch(key: "users")
        }
    }
    
    func getUser(id: String) async throws -> User {
        try await remoteDataSource.fetch("users/\(id)")
    }
    
    func createUser(_ user: User) async throws -> User {
        let createdUser: User = try await remoteDataSource.post("users", body: user)
        
        // Update local cache
        var users: [User] = (try? await localDataSource.fetch(key: "users")) ?? []
        users.append(createdUser)
        try await localDataSource.save(users, key: "users")
        
        return createdUser
    }
    
    func updateUser(_ user: User) async throws -> User {
        let updatedUser: User = try await remoteDataSource.put("users/\(user.id)", body: user)
        
        // Update local cache
        var users: [User] = try await localDataSource.fetch(key: "users")
        if let index = users.firstIndex(where: { $0.id == user.id }) {
            users[index] = updatedUser
            try await localDataSource.save(users, key: "users")
        }
        
        return updatedUser
    }
    
    func deleteUser(id: String) async throws {
        try await remoteDataSource.delete("users/\(id)")
        
        // Update local cache
        var users: [User] = try await localDataSource.fetch(key: "users")
        users.removeAll { $0.id == id }
        try await localDataSource.save(users, key: "users")
    }
    
    func searchUsers(query: String) async throws -> [User] {
        let users: [User] = try await remoteDataSource.fetch("users/search?q=\(query)")
        return users
    }
}
```

### Repository con DTOs

```swift
// Data Transfer Object
struct UserDTO: Codable {
    let id: String
    let name: String
    let email: String
    
    func toDomain() -> User {
        User(id: id, name: name, email: email)
    }
}

extension User {
    func toDTO() -> UserDTO {
        UserDTO(id: id, name: name, email: email)
    }
}

class UserRepositoryWithDTO: UserRepository {
    private let remote: RemoteDataSource
    private let local: LocalDataSource
    
    init(remote: RemoteDataSource, local: LocalDataSource) {
        self.remote = remote
        self.local = local
    }
    
    func fetchUsers() async throws -> [User] {
        let dtos: [UserDTO] = try await remote.fetch("users")
        let users = dtos.map { $0.toDomain() }
        
        // Cache DTOs
        try await local.save(dtos, key: "users")
        
        return users
    }
    
    func createUser(_ user: User) async throws -> User {
        let dto = user.toDTO()
        let createdDTO: UserDTO = try await remote.post("users", body: dto)
        return createdDTO.toDomain()
    }
}
```

---

## Caching Strategies

### Cache-First Strategy

```swift
class CacheFirstRepository: UserRepository {
    private let remote: RemoteDataSource
    private let local: LocalDataSource
    private let cacheTimeout: TimeInterval = 3600 // 1 hour
    
    func fetchUsers() async throws -> [User] {
        // 1. Check local first
        if let cachedUsers = try? await local.fetch(key: "users") as [User],
           !isCacheExpired() {
            return cachedUsers
        }
        
        // 2. Fetch from remote
        let users: [User] = try await remote.fetch("users")
        
        // 3. Update cache
        try await local.save(users, key: "users")
        updateCacheTimestamp()
        
        return users
    }
    
    private func isCacheExpired() -> Bool {
        guard let lastUpdate = UserDefaults.standard.object(forKey: "users_last_update") as? Date else {
            return true
        }
        return Date().timeIntervalSince(lastUpdate) > cacheTimeout
    }
    
    private func updateCacheTimestamp() {
        UserDefaults.standard.set(Date(), forKey: "users_last_update")
    }
}
```

### Network-First Strategy

```swift
class NetworkFirstRepository: UserRepository {
    private let remote: RemoteDataSource
    private let local: LocalDataSource
    
    func fetchUsers() async throws -> [User] {
        do {
            // 1. Try network first
            let users: [User] = try await remote.fetch("users")
            
            // 2. Cache result
            try await local.save(users, key: "users")
            
            return users
        } catch {
            // 3. Fallback to cache
            return try await local.fetch(key: "users")
        }
    }
}
```

### Hybrid Strategy

```swift
class HybridRepository: UserRepository {
    private let remote: RemoteDataSource
    private let local: LocalDataSource
    
    func fetchUsers() async throws -> [User] {
        // Return cache immediately
        let cachedUsers: [User]? = try? await local.fetch(key: "users")
        
        // Fetch fresh data in background
        Task {
            let users: [User] = try await remote.fetch("users")
            try await local.save(users, key: "users")
        }
        
        return cachedUsers ?? []
    }
}
```

---

## Error Handling

### Repository Errors

```swift
enum RepositoryError: Error {
    case networkError(Error)
    case decodingError(Error)
    case notFound
    case unauthorized
    case serverError(Int)
    case cacheError
    
    var localizedDescription: String {
        switch self {
        case .networkError(let error):
            return "Network error: \(error.localizedDescription)"
        case .decodingError(let error):
            return "Decoding error: \(error.localizedDescription)"
        case .notFound:
            return "Resource not found"
        case .unauthorized:
            return "Unauthorized access"
        case .serverError(let code):
            return "Server error: \(code)"
        case .cacheError:
            return "Cache error"
        }
    }
}

class SafeUserRepository: UserRepository {
    private let remote: RemoteDataSource
    private let local: LocalDataSource
    
    func fetchUsers() async throws -> [User] {
        do {
            let users: [User] = try await remote.fetch("users")
            try await local.save(users, key: "users")
            return users
        } catch let error as URLError {
            throw RepositoryError.networkError(error)
        } catch let error as DecodingError {
            throw RepositoryError.decodingError(error)
        } catch {
            // Try cache
            do {
                return try await local.fetch(key: "users")
            } catch {
                throw RepositoryError.cacheError
            }
        }
    }
}
```

---

## Testing

### Unit Testing Repository

```swift
import XCTest
@testable import MyApp

class UserRepositoryTests: XCTestCase {
    var repository: UserRepository!
    var mockRemote: MockRemoteDataSource!
    var mockLocal: MockLocalDataSource!
    
    override func setUp() {
        mockRemote = MockRemoteDataSource()
        mockLocal = MockLocalDataSource()
        repository = UserRepositoryImpl(
            remoteDataSource: mockRemote,
            localDataSource: mockLocal
        )
    }
    
    func testFetchUsers_Success() async throws {
        // Given
        let expectedUsers = [
            User(id: "1", name: "Test User", email: "test@example.com")
        ]
        mockRemote.usersToReturn = expectedUsers
        
        // When
        let users = try await repository.fetchUsers()
        
        // Then
        XCTAssertEqual(users.count, 1)
        XCTAssertEqual(users.first?.name, "Test User")
    }
    
    func testFetchUsers_NetworkError_ReturnsCached() async throws {
        // Given
        let cachedUsers = [
            User(id: "2", name: "Cached User", email: "cached@example.com")
        ]
        mockRemote.shouldFail = true
        mockLocal.cachedUsers = cachedUsers
        
        // When
        let users = try await repository.fetchUsers()
        
        // Then
        XCTAssertEqual(users.count, 1)
        XCTAssertEqual(users.first?.name, "Cached User")
    }
}

// MARK: - Mocks

class MockRemoteDataSource: RemoteDataSource {
    var usersToReturn: [User] = []
    var shouldFail = false
    
    func fetch<T: Decodable>(_ endpoint: String) async throws -> T {
        if shouldFail {
            throw URLError(.notConnectedToInternet)
        }
        return usersToReturn as! T
    }
    
    func post<T: Encodable, R: Decodable>(_ endpoint: String, body: T) async throws -> R {
        return usersToReturn.first as! R
    }
    
    func put<T: Encodable, R: Decodable>(_ endpoint: String, body: T) async throws -> R {
        return body as! R
    }
    
    func delete(_ endpoint: String) async throws {
        // Do nothing
    }
}

class MockLocalDataSource: LocalDataSource {
    var cachedUsers: [User] = []
    
    func save<T: Encodable>(_ items: [T], key: String) async throws {
        cachedUsers = items as? [User] ?? []
    }
    
    func fetch<T: Decodable>(key: String) async throws -> [T] {
        return cachedUsers as? [T] ?? []
    }
    
    func delete(key: String) async throws {
        cachedUsers = []
    }
}
```

---

## Casos de Uso Reales

### E-Commerce Repository

```swift
protocol ProductRepository {
    func fetchProducts() async throws -> [Product]
    func fetchProduct(id: String) async throws -> Product
    func searchProducts(query: String) async throws -> [Product]
    func fetchProductsByCategory(_ category: String) async throws -> [Product]
}

class ProductRepositoryImpl: ProductRepository {
    private let remote: RemoteDataSource
    private let local: LocalDataSource
    private let cache: NSCache<NSString, CacheEntry>
    
    init(remote: RemoteDataSource, local: LocalDataSource) {
        self.remote = remote
        self.local = local
        self.cache = NSCache()
    }
    
    func fetchProducts() async throws -> [Product] {
        // Check memory cache
        if let cached = cache.object(forKey: "all_products") {
            if !cached.isExpired {
                return cached.products
            }
        }
        
        // Fetch from remote
        let products: [Product] = try await remote.fetch("products")
        
        // Update caches
        let entry = CacheEntry(products: products, timestamp: Date())
        cache.setObject(entry, forKey: "all_products")
        try await local.save(products, key: "products")
        
        return products
    }
    
    func searchProducts(query: String) async throws -> [Product] {
        try await remote.fetch("products/search?q=\(query)")
    }
    
    func fetchProductsByCategory(_ category: String) async throws -> [Product] {
        try await remote.fetch("products/category/\(category)")
    }
}

class CacheEntry {
    let products: [Product]
    let timestamp: Date
    let lifetime: TimeInterval = 300 // 5 minutes
    
    init(products: [Product], timestamp: Date) {
        self.products = products
        self.timestamp = timestamp
    }
    
    var isExpired: Bool {
        Date().timeIntervalSince(timestamp) > lifetime
    }
}
```

---

## Best Practices

### 1. Interface Segregation

```swift
// ✅ Bueno - Interfaces específicas
protocol UserReadRepository {
    func fetchUsers() async throws -> [User]
    func getUser(id: String) async throws -> User
}

protocol UserWriteRepository {
    func createUser(_ user: User) async throws -> User
    func updateUser(_ user: User) async throws -> User
    func deleteUser(id: String) async throws
}
```

### 2. Dependency Injection

```swift
// ✅ Bueno - Inyección de dependencias
class UserRepository {
    private let remote: RemoteDataSource
    private let local: LocalDataSource
    
    init(remote: RemoteDataSource, local: LocalDataSource) {
        self.remote = remote
        self.local = local
    }
}
```

### 3. Error Handling Consistente

```swift
// ✅ Bueno - Errores específicos del repository
enum UserRepositoryError: Error {
    case userNotFound
    case invalidUserData
    case networkFailure
}
```

---

## Checklist de Aprendizaje

### Básico
- [ ] Entender Repository Pattern
- [ ] Crear Repository Protocol
- [ ] Implementar Data Sources básicos
- [ ] Separar Remote y Local

### Intermedio
- [ ] Caching strategies
- [ ] Error handling robusto
- [ ] DTOs y mappers
- [ ] Testing con mocks

### Avanzado
- [ ] Generic repositories
- [ ] Hybrid caching
- [ ] Performance optimization
- [ ] Offline-first strategies

---

## Próximos Pasos

### 1. Patterns Avanzados
- Unit of Work
- Specification Pattern
- Query Object

### 2. Optimizaciones
- Pagination
- Batch operations
- Real-time sync

---

## Recursos

### Documentación
- 📚 [Repository Pattern](https://martinfowler.com/eaaCatalog/repository.html)
- 📖 [Data Access Layer](https://developer.apple.com/documentation/swift)

### Videos
- 🎥 [Repository Pattern iOS](https://www.youtube.com/watch?v=4GjXq2Sr55Q)
- 🎥 [Clean Architecture Data Layer](https://www.youtube.com/watch?v=o_TH-Y78tt4)

### Artículos
- 📝 [Repository Pattern in Swift](https://www.swiftbysundell.com/articles/data-layer-in-swift/)
- 📝 [iOS Architecture Patterns](https://medium.com/ios-os-x-development/repository-pattern-in-swift)

---

**Anterior:** [Clean Architecture](Clean-Architecture.md)  
**Próximo:** [Use Cases](Use-Cases.md)
