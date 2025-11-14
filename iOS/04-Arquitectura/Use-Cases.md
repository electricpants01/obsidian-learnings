# Use Cases en iOS - Guía Completa

## Tabla de Contenido
1. [¿Qué son Use Cases?](#qué-son-use-cases)
2. [Ventajas de Use Cases](#ventajas-de-use-cases)
3. [Estructura de un Use Case](#estructura-de-un-use-case)
4. [Use Cases Básicos](#use-cases-básicos)
5. [Use Cases Complejos](#use-cases-complejos)
6. [Input y Output](#input-y-output)
7. [Composición de Use Cases](#composición-de-use-cases)
8. [Error Handling](#error-handling)
9. [Testing Use Cases](#testing-use-cases)
10. [Casos de Uso Reales](#casos-de-uso-reales)
11. [Best Practices](#best-practices)
12. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
13. [Próximos Pasos](#próximos-pasos)
14. [Recursos](#recursos)

---

## ¿Qué son Use Cases?

Los **Use Cases** (Casos de Uso) encapsulan la lógica de negocio de la aplicación en una sola responsabilidad. Cada Use Case representa una acción que el usuario puede realizar.

### Concepto

```
┌────────────────────────────────────────┐
│         Presentation Layer             │
│         (ViewModel/View)               │
│              ↓ Llama                   │
└────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────┐
│           USE CASE                     │
│   - Una sola responsabilidad           │
│   - Lógica de negocio                  │
│   - Orquesta Repositories              │
│              ↓ Usa                     │
└────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────┐
│         Repositories                   │
│    (Data Access Layer)                 │
└────────────────────────────────────────┘
```

### Principios

```swift
// ✅ Principios de Use Cases

// 1. Single Responsibility
// - Un Use Case = Una acción
// - Fácil de entender y mantener

// 2. Business Logic
// - Contiene reglas de negocio
// - Independiente de frameworks

// 3. Reusability
// - Se pueden reutilizar en diferentes partes
// - ViewModels, Background Tasks, etc.

// 4. Testability
// - Fácil de testear sin UI
// - Mocks de reposit ories
```

---

## Ventajas de Use Cases

### 1. Separación de Responsabilidades

```swift
// ❌ Sin Use Cases - ViewModel con toda la lógica
class UserViewModel {
    func loadUsers() async {
        // Fetch from API
        // Validate data
        // Transform data
        // Cache locally
        // Update UI
        // All in one place!
    }
}

// ✅ Con Use Cases - Separación clara
class UserViewModel {
    private let fetchUsersUseCase: FetchUsersUseCase
    
    func loadUsers() async {
        let users = try await fetchUsersUseCase.execute()
        // Solo actualizar UI
    }
}
```

### 2. Reusabilidad

```swift
// Use Case puede usarse en múltiples lugares
class FetchUsersUseCase {
    func execute() async throws -> [User] {
        // Logic
    }
}

// En ViewModel
class UserListViewModel {
    let fetchUsers = FetchUsersUseCase()
}

// En Background Task
class SyncTask {
    let fetchUsers = FetchUsersUseCase()
}

// En Widget Extension
class UserWidget {
    let fetchUsers = FetchUsersUseCase()
}
```

### 3. Testabilidad

```swift
// Fácil de testear sin UI
class FetchUsersUseCaseTests: XCTestCase {
    func testExecute() async throws {
        let useCase = FetchUsersUseCase(
            repository: MockUserRepository()
        )
        
        let users = try await useCase.execute()
        
        XCTAssertFalse(users.isEmpty)
    }
}
```

---

## Estructura de un Use Case

### Protocol Básico

```swift
protocol UseCase {
    associatedtype Input
    associatedtype Output
    
    func execute(_ input: Input) async throws -> Output
}
```

### Implementation Simple

```swift
// Use Case sin parámetros
struct FetchUsersUseCase: UseCase {
    typealias Input = Void
    typealias Output = [User]
    
    private let repository: UserRepository
    
    init(repository: UserRepository) {
        self.repository = repository
    }
    
    func execute(_ input: Void = ()) async throws -> [User] {
        try await repository.fetchUsers()
    }
}
```

### Implementation con Input

```swift
// Use Case con parámetros
struct LoginUseCase: UseCase {
    struct Input {
        let email: String
        let password: String
    }
    
    typealias Output = User
    
    private let authRepository: AuthRepository
    
    init(authRepository: AuthRepository) {
        self.authRepository = authRepository
    }
    
    func execute(_ input: Input) async throws -> User {
        // Validar
        guard input.email.contains("@") else {
            throw ValidationError.invalidEmail
        }
        
        guard input.password.count >= 6 else {
            throw ValidationError.weakPassword
        }
        
        // Ejecutar
        return try await authRepository.login(
            email: input.email,
            password: input.password
        )
    }
}

enum ValidationError: Error {
    case invalidEmail
    case weakPassword
}
```

---

## Use Cases Básicos

### Fetch Data

```swift
protocol FetchProductsUseCase {
    func execute() async throws -> [Product]
}

class FetchProductsUseCaseImpl: FetchProductsUseCase {
    private let repository: ProductRepository
    
    init(repository: ProductRepository) {
        self.repository = repository
    }
    
    func execute() async throws -> [Product] {
        try await repository.fetchProducts()
    }
}
```

### Create Entity

```swift
protocol CreateUserUseCase {
    func execute(name: String, email: String) async throws -> User
}

class CreateUserUseCaseImpl: CreateUserUseCase {
    private let repository: UserRepository
    private let validator: UserValidator
    
    init(
        repository: UserRepository,
        validator: UserValidator = UserValidator()
    ) {
        self.repository = repository
        self.validator = validator
    }
    
    func execute(name: String, email: String) async throws -> User {
        // Validate
        try validator.validateName(name)
        try validator.validateEmail(email)
        
        // Create
        let user = User(
            id: UUID().uuidString,
            name: name,
            email: email
        )
        
        // Save
        return try await repository.createUser(user)
    }
}

class UserValidator {
    func validateName(_ name: String) throws {
        guard !name.isEmpty else {
            throw ValidationError.emptyName
        }
        guard name.count >= 3 else {
            throw ValidationError.nameTooShort
        }
    }
    
    func validateEmail(_ email: String) throws {
        guard email.contains("@") && email.contains(".") else {
            throw ValidationError.invalidEmail
        }
    }
}

enum ValidationError: Error {
    case emptyName
    case nameTooShort
    case invalidEmail
}
```

### Update Entity

```swift
protocol UpdateUserUseCase {
    func execute(_ user: User) async throws -> User
}

class UpdateUserUseCaseImpl: UpdateUserUseCase {
    private let repository: UserRepository
    
    init(repository: UserRepository) {
        self.repository = repository
    }
    
    func execute(_ user: User) async throws -> User {
        // Validate
        guard user.isValid() else {
            throw UserError.invalidUser
        }
        
        // Update
        return try await repository.updateUser(user)
    }
}
```

### Delete Entity

```swift
protocol DeleteUserUseCase {
    func execute(userId: String) async throws
}

class DeleteUserUseCaseImpl: DeleteUserUseCase {
    private let repository: UserRepository
    
    init(repository: UserRepository) {
        self.repository = repository
    }
    
    func execute(userId: String) async throws {
        try await repository.deleteUser(id: userId)
    }
}
```

---

## Use Cases Complejos

### Use Case con Múltiples Repositories

```swift
protocol CreateOrderUseCase {
    func execute(userId: String, items: [CartItem]) async throws -> Order
}

class CreateOrderUseCaseImpl: CreateOrderUseCase {
    private let orderRepository: OrderRepository
    private let productRepository: ProductRepository
    private let userRepository: UserRepository
    
    init(
        orderRepository: OrderRepository,
        productRepository: ProductRepository,
        userRepository: UserRepository
    ) {
        self.orderRepository = orderRepository
        self.productRepository = productRepository
        self.userRepository = userRepository
    }
    
    func execute(userId: String, items: [CartItem]) async throws -> Order {
        // 1. Validate user
        let user = try await userRepository.getUser(id: userId)
        guard user.isActive else {
            throw OrderError.inactiveUser
        }
        
        // 2. Validate products and stock
        for item in items {
            let product = try await productRepository.getProduct(id: item.productId)
            
            guard product.isAvailable() else {
                throw OrderError.productNotAvailable(product.name)
            }
            
            guard product.stock >= item.quantity else {
                throw OrderError.insufficientStock(product.name)
            }
        }
        
        // 3. Calculate total
        let total = try await calculateTotal(items: items)
        
        // 4. Create order
        let order = Order(
            id: UUID().uuidString,
            userId: userId,
            items: items.map { OrderItem(from: $0) },
            total: total,
            status: .pending,
            createdAt: Date()
        )
        
        // 5. Save order
        let savedOrder = try await orderRepository.saveOrder(order)
        
        // 6. Update stock
        for item in items {
            try await productRepository.updateStock(
                productId: item.productId,
                quantity: -item.quantity
            )
        }
        
        return savedOrder
    }
    
    private func calculateTotal(items: [CartItem]) async throws -> Double {
        var total = 0.0
        for item in items {
            let product = try await productRepository.getProduct(id: item.productId)
            total += product.price * Double(item.quantity)
        }
        return total
    }
}

enum OrderError: Error {
    case inactiveUser
    case productNotAvailable(String)
    case insufficientStock(String)
    case invalidTotal
}
```

### Use Case con Lógica de Negocio Compleja

```swift
protocol ProcessPaymentUseCase {
    func execute(orderId: String, paymentMethod: PaymentMethod) async throws -> Payment
}

class ProcessPaymentUseCaseImpl: ProcessPaymentUseCase {
    private let orderRepository: OrderRepository
    private let paymentGateway: PaymentGateway
    private let emailService: EmailService
    private let analyticsService: AnalyticsService
    
    init(
        orderRepository: OrderRepository,
        paymentGateway: PaymentGateway,
        emailService: EmailService,
        analyticsService: AnalyticsService
    ) {
        self.orderRepository = orderRepository
        self.paymentGateway = paymentGateway
        self.emailService = emailService
        self.analyticsService = analyticsService
    }
    
    func execute(orderId: String, paymentMethod: PaymentMethod) async throws -> Payment {
        // 1. Get order
        let order = try await orderRepository.getOrder(id: orderId)
        
        // 2. Validate order status
        guard order.status == .pending else {
            throw PaymentError.invalidOrderStatus
        }
        
        // 3. Apply discounts if applicable
        let finalAmount = applyDiscounts(to: order)
        
        // 4. Process payment
        let payment = try await paymentGateway.charge(
            amount: finalAmount,
            method: paymentMethod
        )
        
        // 5. Update order status
        try await orderRepository.updateOrderStatus(
            id: orderId,
            status: .processing
        )
        
        // 6. Send confirmation email
        try? await emailService.sendOrderConfirmation(order: order)
        
        // 7. Track analytics
        await analyticsService.trackPurchase(
            orderId: orderId,
            amount: finalAmount
        )
        
        return payment
    }
    
    private func applyDiscounts(to order: Order) -> Double {
        var amount = order.total
        
        // Apply any business rules for discounts
        if order.items.count > 5 {
            amount *= 0.9 // 10% discount
        }
        
        return amount
    }
}

enum PaymentError: Error {
    case invalidOrderStatus
    case paymentDeclined
    case insufficientFunds
}
```

---

## Input y Output

### Input DTO

```swift
struct CreateUserInput {
    let name: String
    let email: String
    let age: Int
    let address: Address?
    
    struct Address {
        let street: String
        let city: String
        let zipCode: String
    }
}

protocol CreateUserUseCase {
    func execute(_ input: CreateUserInput) async throws -> User
}
```

### Output DTO

```swift
struct LoginOutput {
    let user: User
    let accessToken: String
    let refreshToken: String
    let expiresAt: Date
}

protocol LoginUseCase {
    func execute(email: String, password: String) async throws -> LoginOutput
}
```

### Result Type

```swift
enum UseCaseResult<T> {
    case success(T)
    case failure(Error)
}

protocol FetchDataUseCase {
    func execute() async -> UseCaseResult<[Data]>
}

class FetchDataUseCaseImpl: FetchDataUseCase {
    func execute() async -> UseCaseResult<[Data]> {
        do {
            let data = try await repository.fetchData()
            return .success(data)
        } catch {
            return .failure(error)
        }
    }
}
```

---

## Composición de Use Cases

### Use Case que usa otro Use Case

```swift
protocol CheckoutUseCase {
    func execute(userId: String, cartId: String) async throws -> CheckoutResult
}

class CheckoutUseCaseImpl: CheckoutUseCase {
    private let getCartUseCase: GetCartUseCase
    private let createOrderUseCase: CreateOrderUseCase
    private let processPaymentUseCase: ProcessPaymentUseCase
    private let clearCartUseCase: ClearCartUseCase
    
    init(
        getCartUseCase: GetCartUseCase,
        createOrderUseCase: CreateOrderUseCase,
        processPaymentUseCase: ProcessPaymentUseCase,
        clearCartUseCase: ClearCartUseCase
    ) {
        self.getCartUseCase = getCartUseCase
        self.createOrderUseCase = createOrderUseCase
        self.processPaymentUseCase = processPaymentUseCase
        self.clearCartUseCase = clearCartUseCase
    }
    
    func execute(userId: String, cartId: String) async throws -> CheckoutResult {
        // 1. Get cart
        let cart = try await getCartUseCase.execute(cartId: cartId)
        
        guard !cart.items.isEmpty else {
            throw CheckoutError.emptyCart
        }
        
        // 2. Create order
        let order = try await createOrderUseCase.execute(
            userId: userId,
            items: cart.items
        )
        
        // 3. Process payment
        let payment = try await processPaymentUseCase.execute(
            orderId: order.id,
            paymentMethod: cart.paymentMethod
        )
        
        // 4. Clear cart
        try await clearCartUseCase.execute(cartId: cartId)
        
        return CheckoutResult(
            order: order,
            payment: payment
        )
    }
}

struct CheckoutResult {
    let order: Order
    let payment: Payment
}

enum CheckoutError: Error {
    case emptyCart
    case invalidPaymentMethod
}
```

---

## Error Handling

### Custom Errors

```swift
enum UserUseCaseError: Error {
    case userNotFound
    case invalidCredentials
    case emailAlreadyExists
    case weakPassword
    case networkError
    
    var localizedDescription: String {
        switch self {
        case .userNotFound:
            return "User not found"
        case .invalidCredentials:
            return "Invalid email or password"
        case .emailAlreadyExists:
            return "Email already registered"
        case .weakPassword:
            return "Password must be at least 8 characters"
        case .networkError:
            return "Network connection failed"
        }
    }
}
```

### Error Mapping

```swift
class SignUpUseCase {
    func execute(email: String, password: String) async throws -> User {
        do {
            // Validate
            try validateEmail(email)
            try validatePassword(password)
            
            // Create user
            return try await repository.createUser(email: email, password: password)
        } catch let error as ValidationError {
            throw UserUseCaseError.weakPassword
        } catch let error as RepositoryError {
            if case .conflict = error {
                throw UserUseCaseError.emailAlreadyExists
            }
            throw UserUseCaseError.networkError
        }
    }
}
```

---

## Testing Use Cases

### Basic Unit Test

```swift
import XCTest
@testable import MyApp

class LoginUseCaseTests: XCTestCase {
    var useCase: LoginUseCase!
    var mockRepository: MockAuthRepository!
    
    override func setUp() {
        mockRepository = MockAuthRepository()
        useCase = LoginUseCaseImpl(authRepository: mockRepository)
    }
    
    func testExecute_ValidCredentials_ReturnsUser() async throws {
        // Given
        let email = "test@example.com"
        let password = "password123"
        let expectedUser = User(id: "1", name: "Test", email: email)
        mockRepository.userToReturn = expectedUser
        
        // When
        let input = LoginUseCase.Input(email: email, password: password)
        let user = try await useCase.execute(input)
        
        // Then
        XCTAssertEqual(user.id, expectedUser.id)
        XCTAssertEqual(user.email, email)
    }
    
    func testExecute_InvalidEmail_ThrowsError() async {
        // Given
        let email = "invalid-email"
        let password = "password123"
        
        // When/Then
        let input = LoginUseCase.Input(email: email, password: password)
        
        do {
            _ = try await useCase.execute(input)
            XCTFail("Should throw validation error")
        } catch {
            XCTAssertTrue(error is ValidationError)
        }
    }
    
    func testExecute_WeakPassword_ThrowsError() async {
        // Given
        let email = "test@example.com"
        let password = "123"
        
        // When/Then
        let input = LoginUseCase.Input(email: email, password: password)
        
        do {
            _ = try await useCase.execute(input)
            XCTFail("Should throw validation error")
        } catch ValidationError.weakPassword {
            // Success
        } catch {
            XCTFail("Wrong error type")
        }
    }
}

// MARK: - Mock

class MockAuthRepository: AuthRepository {
    var userToReturn: User?
    var shouldFail = false
    
    func login(email: String, password: String) async throws -> User {
        if shouldFail {
            throw RepositoryError.networkError
        }
        guard let user = userToReturn else {
            throw RepositoryError.notFound
        }
        return user
    }
}
```

### Testing Complex Use Cases

```swift
class CreateOrderUseCaseTests: XCTestCase {
    var useCase: CreateOrderUseCase!
    var mockOrderRepo: MockOrderRepository!
    var mockProductRepo: MockProductRepository!
    var mockUserRepo: MockUserRepository!
    
    override func setUp() {
        mockOrderRepo = MockOrderRepository()
        mockProductRepo = MockProductRepository()
        mockUserRepo = MockUserRepository()
        
        useCase = CreateOrderUseCaseImpl(
            orderRepository: mockOrderRepo,
            productRepository: mockProductRepo,
            userRepository: mockUserRepo
        )
    }
    
    func testExecute_ValidOrder_CreatesSuccessfully() async throws {
        // Given
        let user = User(id: "1", name: "Test", email: "test@example.com", isActive: true)
        let product = Product(id: "p1", name: "Product", price: 10.0, stock: 5)
        let items = [CartItem(productId: "p1", quantity: 2)]
        
        mockUserRepo.users = [user]
        mockProductRepo.products = [product]
        
        // When
        let order = try await useCase.execute(userId: "1", items: items)
        
        // Then
        XCTAssertEqual(order.userId, "1")
        XCTAssertEqual(order.items.count, 1)
        XCTAssertEqual(order.total, 20.0)
        XCTAssertTrue(mockOrderRepo.savedOrders.contains(where: { $0.id == order.id }))
    }
    
    func testExecute_InsufficientStock_ThrowsError() async {
        // Given
        let user = User(id: "1", name: "Test", email: "test@example.com", isActive: true)
        let product = Product(id: "p1", name: "Product", price: 10.0, stock: 1)
        let items = [CartItem(productId: "p1", quantity: 5)] // More than stock
        
        mockUserRepo.users = [user]
        mockProductRepo.products = [product]
        
        // When/Then
        do {
            _ = try await useCase.execute(userId: "1", items: items)
            XCTFail("Should throw insufficient stock error")
        } catch OrderError.insufficientStock {
            // Success
        } catch {
            XCTFail("Wrong error type")
        }
    }
}
```

---

## Casos de Uso Reales

### User Authentication Flow

```swift
// 1. Register
protocol RegisterUseCase {
    func execute(email: String, password: String, name: String) async throws -> User
}

// 2. Login
protocol LoginUseCase {
    func execute(email: String, password: String) async throws -> LoginOutput
}

// 3. Logout
protocol LogoutUseCase {
    func execute() async throws
}

// 4. Reset Password
protocol ResetPasswordUseCase {
    func execute(email: String) async throws
}
```

### E-Commerce Flow

```swift
// 1. Browse Products
protocol FetchProductsUseCase {
    func execute(category: String?) async throws -> [Product]
}

// 2. Add to Cart
protocol AddToCartUseCase {
    func execute(productId: String, quantity: Int) async throws
}

// 3. Checkout
protocol CheckoutUseCase {
    func execute(userId: String, cartId: String) async throws -> CheckoutResult
}

// 4. Track Order
protocol TrackOrderUseCase {
    func execute(orderId: String) async throws -> OrderStatus
}
```

---

## Best Practices

### 1. Single Responsibility

```swift
// ✅ Bueno - Un Use Case, una acción
class FetchUsersUseCase { }
class CreateUserUseCase { }
class DeleteUserUseCase { }

// ❌ Malo - Use Case con múltiples acciones
class UserUseCase {
    func fetch() { }
    func create() { }
    func delete() { }
}
```

### 2. Dependency Injection

```swift
// ✅ Bueno - Inyección de dependencias
class UseCase {
    private let repository: Repository
    
    init(repository: Repository) {
        self.repository = repository
    }
}
```

### 3. Immutable Input/Output

```swift
// ✅ Bueno - Input inmutable
struct CreateUserInput {
    let name: String
    let email: String
}
```

---

## Checklist de Aprendizaje

### Básico
- [ ] Entender qué son Use Cases
- [ ] Crear Use Cases simples
- [ ] Usar protocols para Use Cases
- [ ] Separar lógica de negocio

### Intermedio
- [ ] Use Cases complejos
- [ ] Composición de Use Cases
- [ ] Error handling robusto
- [ ] Testing de Use Cases

### Avanzado
- [ ] Use Cases con múltiples repositorios
- [ ] Transaction management
- [ ] Saga pattern
- [ ] Event sourcing

---

## Próximos Pasos

### 1. Patterns Avanzados
- Command Pattern
- Chain of Responsibility
- Saga Pattern

### 2. Optimizaciones
- Caching en Use Cases
- Retry logic
- Circuit breaker

---

## Recursos

### Documentación
- 📚 [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- 📖 [Use Cases](https://martinfowler.com/bliki/UseCase.html)

### Videos
- 🎥 [Use Cases in iOS](https://www.youtube.com/watch?v=o_TH-Y78tt4)
- 🎥 [Clean Architecture Use Cases](https://www.youtube.com/watch?v=4GjXq2Sr55Q)

### Artículos
- 📝 [Use Cases in Swift](https://www.swiftbysundell.com/articles/use-cases-in-swift/)
- 📝 [Business Logic Layer](https://medium.com/ios-development/use-cases-in-ios)

---

**Anterior:** [Repository Pattern](Repository-Pattern.md)  
**Próximo:** [Actors](../05-Concurrency/Actors.md)
