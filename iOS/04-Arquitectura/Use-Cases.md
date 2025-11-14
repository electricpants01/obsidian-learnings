# Use Cases

## ¿Qué son Use Cases?

Encapsulan la lógica de negocio de una acción específica.

```swift
// Protocol
protocol UseCase {
    associatedtype Input
    associatedtype Output
    
    func execute(_ input: Input) async throws -> Output
}

// Fetch Users Use Case
struct FetchUsersUseCase: UseCase {
    typealias Input = Void
    typealias Output = [User]
    
    private let repository: UserRepository
    
    init(repository: UserRepository) {
        self.repository = repository
    }
    
    func execute(_ input: Void) async throws -> [User] {
        try await repository.fetchUsers()
    }
}

// Login Use Case
struct LoginUseCase: UseCase {
    struct Input {
        let email: String
        let password: String
    }
    
    typealias Output = User
    
    private let authRepository: AuthRepository
    
    func execute(_ input: Input) async throws -> User {
        guard input.email.contains("@") else {
            throw ValidationError.invalidEmail
        }
        
        guard input.password.count >= 6 else {
            throw ValidationError.weakPassword
        }
        
        return try await authRepository.login(
            email: input.email,
            password: input.password
        )
    }
}
```

## Use Case con Validación

```swift
struct CreateUserUseCase: UseCase {
    struct Input {
        let name: String
        let email: String
        let age: Int
    }
    
    typealias Output = User
    
    private let repository: UserRepository
    private let validator: UserValidator
    
    func execute(_ input: Input) async throws -> User {
        // Validar
        try validator.validate(name: input.name)
        try validator.validate(email: input.email)
        try validator.validate(age: input.age)
        
        // Crear
        let user = User(
            id: UUID().uuidString,
            name: input.name,
            email: input.email
        )
        
        // Guardar
        try await repository.saveUser(user)
        
        return user
    }
}
```

## Uso en ViewModel

```swift
@MainActor
class LoginViewModel: ObservableObject {
    @Published var email = ""
    @Published var password = ""
    @Published var isLoading = false
    @Published var error: String?
    
    private let loginUseCase: LoginUseCase
    
    init(loginUseCase: LoginUseCase) {
        self.loginUseCase = loginUseCase
    }
    
    func login() async {
        isLoading = true
        error = nil
        
        do {
            let input = LoginUseCase.Input(
                email: email,
                password: password
            )
            let user = try await loginUseCase.execute(input)
            // Handle success
        } catch {
            self.error = error.localizedDescription
        }
        
        isLoading = false
    }
}
```

## Recursos

- 📚 [Use Cases](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
