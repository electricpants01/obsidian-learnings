# Clean Architecture en iOS

## Capas de Clean Architecture

```
Presentation ←→ Domain ←→ Data
    (UI)      (UseCases)  (Repository)
```

## Domain Layer

```swift
// Entity
struct User {
    let id: String
    let name: String
    let email: String
}

// Use Case Protocol
protocol FetchUsersUseCase {
    func execute() async throws -> [User]
}

// Use Case Implementation
class FetchUsersUseCaseImpl: FetchUsersUseCase {
    private let repository: UserRepository
    
    init(repository: UserRepository) {
        self.repository = repository
    }
    
    func execute() async throws -> [User] {
        try await repository.fetchUsers()
    }
}
```

## Data Layer

```swift
// Repository Protocol (en Domain)
protocol UserRepository {
    func fetchUsers() async throws -> [User]
    func saveUser(_ user: User) async throws
}

// Repository Implementation (en Data)
class UserRepositoryImpl: UserRepository {
    private let apiClient: APIClient
    private let database: Database
    
    init(apiClient: APIClient, database: Database) {
        self.apiClient = apiClient
        self.database = database
    }
    
    func fetchUsers() async throws -> [User] {
        // Try cache first
        if let cached = try? await database.getUsers(), !cached.isEmpty {
            return cached
        }
        
        // Fetch from API
        let users = try await apiClient.fetchUsers()
        try await database.saveUsers(users)
        return users
    }
    
    func saveUser(_ user: User) async throws {
        try await apiClient.createUser(user)
        try await database.saveUser(user)
    }
}
```

## Presentation Layer

```swift
@MainActor
class UserListViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var error: String?
    
    private let fetchUsersUseCase: FetchUsersUseCase
    
    init(fetchUsersUseCase: FetchUsersUseCase) {
        self.fetchUsersUseCase = fetchUsersUseCase
    }
    
    func loadUsers() async {
        isLoading = true
        error = nil
        
        do {
            users = try await fetchUsersUseCase.execute()
        } catch {
            self.error = error.localizedDescription
        }
        
        isLoading = false
    }
}
```

## Dependency Injection

```swift
class AppDependencies {
    // Data Layer
    let apiClient = APIClient()
    let database = Database()
    
    // Repositories
    lazy var userRepository: UserRepository = {
        UserRepositoryImpl(apiClient: apiClient, database: database)
    }()
    
    // Use Cases
    lazy var fetchUsersUseCase: FetchUsersUseCase = {
        FetchUsersUseCaseImpl(repository: userRepository)
    }()
    
    // ViewModels
    func makeUserListViewModel() -> UserListViewModel {
        UserListViewModel(fetchUsersUseCase: fetchUsersUseCase)
    }
}
```

## Recursos

- 📚 [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- 🎥 [iOS Clean Architecture](https://www.youtube.com/watch?v=o_TH-Y78tt4)
