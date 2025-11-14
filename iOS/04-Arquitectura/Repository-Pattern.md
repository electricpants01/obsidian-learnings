# Repository Pattern

## ¿Qué es Repository?

Abstrae el acceso a datos, permitiendo cambiar la fuente sin afectar el código.

```swift
// Protocol
protocol UserRepository {
    func fetchUsers() async throws -> [User]
    func getUser(id: String) async throws -> User
    func saveUser(_ user: User) async throws
    func deleteUser(id: String) async throws
}

// Implementation
class UserRepositoryImpl: UserRepository {
    private let remote: RemoteDataSource
    private let local: LocalDataSource
    
    init(remote: RemoteDataSource, local: LocalDataSource) {
        self.remote = remote
        self.local = local
    }
    
    func fetchUsers() async throws -> [User] {
        do {
            let users = try await remote.fetchUsers()
            try await local.saveUsers(users)
            return users
        } catch {
            return try await local.getUsers()
        }
    }
    
    func getUser(id: String) async throws -> User {
        if let cached = try? await local.getUser(id: id) {
            return cached
        }
        let user = try await remote.getUser(id: id)
        try await local.saveUser(user)
        return user
    }
    
    func saveUser(_ user: User) async throws {
        try await remote.saveUser(user)
        try await local.saveUser(user)
    }
    
    func deleteUser(id: String) async throws {
        try await remote.deleteUser(id: id)
        try await local.deleteUser(id: id)
    }
}
```

## Data Sources

```swift
// Remote
class RemoteDataSource {
    func fetchUsers() async throws -> [User] {
        let url = URL(string: "https://api.example.com/users")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode([User].self, from: data)
    }
}

// Local
class LocalDataSource {
    private let context: NSManagedObjectContext
    
    func getUsers() async throws -> [User] {
        let request: NSFetchRequest<UserEntity> = UserEntity.fetchRequest()
        let entities = try context.fetch(request)
        return entities.map { $0.toDomain() }
    }
    
    func saveUsers(_ users: [User]) async throws {
        users.forEach { user in
            let entity = UserEntity(context: context)
            entity.id = user.id
            entity.name = user.name
        }
        try context.save()
    }
}
```

## Recursos

- 📚 [Repository Pattern](https://martinfowler.com/eaaCatalog/repository.html)
