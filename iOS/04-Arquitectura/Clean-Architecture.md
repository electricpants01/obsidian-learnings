# Clean Architecture en iOS - Guía Completa

## Tabla de Contenido
1. [¿Qué es Clean Architecture?](#qué-es-clean-architecture)
2. [Capas de Clean Architecture](#capas-de-clean-architecture)
3. [Presentation Layer](#presentation-layer)
4. [Domain Layer](#domain-layer)
5. [Data Layer](#data-layer)
6. [Dependency Rule](#dependency-rule)
7. [Use Cases](#use-cases)
8. [Entities](#entities)
9. [Repositories](#repositories)
10. [Casos de Uso Completos](#casos-de-uso-completos)
11. [Testing](#testing)
12. [Best Practices](#best-practices)
13. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
14. [Próximos Pasos](#próximos-pasos)
15. [Recursos](#recursos)

---

## ¿Qué es Clean Architecture?

Clean Architecture de Uncle Bob separa el código en capas concéntricas, cada una con responsabilidades específicas.

### Diagrama de Capas

```
┌─────────────────────────────────────────┐
│         Presentation Layer              │
│  (Views, ViewModels, Presenters)       │
│                                         │
│  ┌───────────────────────────────────┐ │
│  │      Domain Layer                 │ │
│  │  (Use Cases, Entities, Protocols) │ │
│  │                                   │ │
│  │  ┌─────────────────────────────┐ │ │
│  │  │    Data Layer               │ │ │
│  │  │  (Repositories, API, DB)    │ │ │
│  │  │                             │ │ │
│  │  │  ┌───────────────────────┐ │ │ │
│  │  │  │  External Frameworks  │ │ │ │
│  │  │  │  (URLSession, Core    │ │ │ │
│  │  │  │   Data, Third Party)  │ │ │ │
│  │  │  └───────────────────────┘ │ │ │
│  │  └─────────────────────────────┘ │ │
│  └───────────────────────────────────┘ │
└─────────────────────────────────────────┘

Dependency Direction: Outward → Inward
```

### Principios

```swift
// ✅ Principios de Clean Architecture

// 1. Independence
// - UI independiente de la lógica de negocio
// - Lógica de negocio independiente de frameworks
// - Testeable sin UI, DB, o frameworks externos

// 2. Dependency Rule
// - Las dependencias apuntan hacia adentro
// - Las capas internas NO conocen las externas
// - Domain es el centro, no depende de nada

// 3. Separation of Concerns
// - Cada capa tiene responsabilidades claras
// - Fácil de mantener y modificar
// - Cambios en una capa no afectan otras
```

---

## Capas de Clean Architecture

### 1. Presentation Layer

```swift
// Vista (SwiftUI)
struct UserListView: View {
    @StateObject private var viewModel: UserListViewModel
    
    init(viewModel: UserListViewModel) {
        _viewModel = StateObject(wrappedValue: viewModel)
    }
    
    var body: some View {
        List(viewModel.users) { user in
            UserRow(user: user)
        }
        .task {
            await viewModel.loadUsers()
        }
    }
}

// ViewModel
@MainActor
class UserListViewModel: ObservableObject {
    @Published var users: [UserPresentation] = []
    @Published var isLoading = false
    @Published var error: String?
    
    private let fetchUsersUseCase: FetchUsersUseCase
    
    init(fetchUsersUseCase: FetchUsersUseCase) {
        self.fetchUsersUseCase = fetchUsersUseCase
    }
    
    func loadUsers() async {
        isLoading = true
        
        do {
            let domainUsers = try await fetchUsersUseCase.execute()
            users = domainUsers.map { UserPresentation(from: $0) }
        } catch {
            self.error = error.localizedDescription
        }
        
        isLoading = false
    }
}

// Presentation Model
struct UserPresentation: Identifiable {
    let id: String
    let displayName: String
    let avatarURL: URL?
    
    init(from domainUser: User) {
        self.id = domainUser.id
        self.displayName = domainUser.name
        self.avatarURL = URL(string: domainUser.avatar ?? "")
    }
}
```

### 2. Domain Layer

```swift
// Entity
struct User {
    let id: String
    let name: String
    let email: String
    let avatar: String?
    
    func isValid() -> Bool {
        !name.isEmpty && email.contains("@")
    }
}

// Use Case Protocol
protocol FetchUsersUseCase {
    func execute() async throws -> [User]
}

// Use Case Implementation
class FetchUsersUseCaseImpl: FetchUsersUseCase {
    private let userRepository: UserRepository
    
    init(userRepository: UserRepository) {
        self.userRepository = userRepository
    }
    
    func execute() async throws -> [User] {
        try await userRepository.fetchUsers()
    }
}

// Repository Protocol (en Domain)
protocol UserRepository {
    func fetchUsers() async throws -> [User]
    func getUser(id: String) async throws -> User
    func saveUser(_ user: User) async throws
    func deleteUser(id: String) async throws
}
```

### 3. Data Layer

```swift
// Repository Implementation
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
        // Try cache first
        if let cachedUsers = try? await localDataSource.getUsers() {
            return cachedUsers.map { $0.toDomain() }
        }
        
        // Fetch from API
        let apiUsers = try await remoteDataSource.fetchUsers()
        let domainUsers = apiUsers.map { $0.toDomain() }
        
        // Cache result
        try? await localDataSource.saveUsers(apiUsers)
        
        return domainUsers
    }
    
    func getUser(id: String) async throws -> User {
        try await remoteDataSource.getUser(id: id).toDomain()
    }
    
    func saveUser(_ user: User) async throws {
        let dto = UserDTO(from: user)
        try await remoteDataSource.saveUser(dto)
        try await localDataSource.saveUser(dto)
    }
    
    func deleteUser(id: String) async throws {
        try await remoteDataSource.deleteUser(id: id)
        try await localDataSource.deleteUser(id: id)
    }
}

// Data Transfer Object (DTO)
struct UserDTO: Codable {
    let id: String
    let name: String
    let email: String
    let avatar: String?
    
    func toDomain() -> User {
        User(id: id, name: name, email: email, avatar: avatar)
    }
    
    init(from domain: User) {
        self.id = domain.id
        self.name = domain.name
        self.email = domain.email
        self.avatar = domain.avatar
    }
}

// Remote Data Source
class RemoteDataSource {
    private let apiClient: APIClient
    
    init(apiClient: APIClient) {
        self.apiClient = apiClient
    }
    
    func fetchUsers() async throws -> [UserDTO] {
        try await apiClient.request("users")
    }
    
    func getUser(id: String) async throws -> UserDTO {
        try await apiClient.request("users/\(id)")
    }
    
    func saveUser(_ user: UserDTO) async throws {
        try await apiClient.post("users", body: user)
    }
    
    func deleteUser(id: String) async throws {
        try await apiClient.delete("users/\(id)")
    }
}

// Local Data Source
class LocalDataSource {
    private let database: Database
    
    init(database: Database) {
        self.database = database
    }
    
    func getUsers() async throws -> [UserDTO] {
        try await database.fetch(UserDTO.self)
    }
    
    func saveUsers(_ users: [UserDTO]) async throws {
        try await database.save(users)
    }
    
    func saveUser(_ user: UserDTO) async throws {
        try await database.save([user])
    }
    
    func deleteUser(id: String) async throws {
        try await database.delete(UserDTO.self, id: id)
    }
}
```

---

## Presentation Layer

### SwiftUI + ViewModel

```swift
// View
struct ProductListView: View {
    @StateObject private var viewModel: ProductListViewModel
    
    init(viewModel: ProductListViewModel) {
        _viewModel = StateObject(wrappedValue: viewModel)
    }
    
    var body: some View {
        NavigationStack {
            content
                .navigationTitle("Products")
                .searchable(text: $viewModel.searchText)
        }
        .task {
            await viewModel.loadProducts()
        }
    }
    
    @ViewBuilder
    private var content: some View {
        if viewModel.isLoading {
            ProgressView()
        } else if let error = viewModel.error {
            ErrorView(message: error) {
                Task { await viewModel.loadProducts() }
            }
        } else {
            productList
        }
    }
    
    private var productList: some View {
        List(viewModel.filteredProducts) { product in
            ProductRow(product: product)
                .onTapGesture {
                    viewModel.selectProduct(product)
                }
        }
    }
}

// ViewModel
@MainActor
class ProductListViewModel: ObservableObject {
    @Published var products: [ProductPresentation] = []
    @Published var searchText = ""
    @Published var isLoading = false
    @Published var error: String?
    
    private let fetchProductsUseCase: FetchProductsUseCase
    private let searchProductsUseCase: SearchProductsUseCase
    
    init(
        fetchProductsUseCase: FetchProductsUseCase,
        searchProductsUseCase: SearchProductsUseCase
    ) {
        self.fetchProductsUseCase = fetchProductsUseCase
        self.searchProductsUseCase = searchProductsUseCase
    }
    
    var filteredProducts: [ProductPresentation] {
        guard !searchText.isEmpty else { return products }
        return products.filter {
            $0.name.localizedCaseInsensitiveContains(searchText)
        }
    }
    
    func loadProducts() async {
        isLoading = true
        error = nil
        
        do {
            let domainProducts = try await fetchProductsUseCase.execute()
            products = domainProducts.map { ProductPresentation(from: $0) }
        } catch {
            self.error = error.localizedDescription
        }
        
        isLoading = false
    }
    
    func selectProduct(_ product: ProductPresentation) {
        print("Selected: \(product.name)")
    }
}

// Presentation Model
struct ProductPresentation: Identifiable {
    let id: String
    let name: String
    let price: String
    let imageURL: URL?
    
    init(from domain: Product) {
        self.id = domain.id
        self.name = domain.name
        self.price = "$\(String(format: "%.2f", domain.price))"
        self.imageURL = URL(string: domain.imageURL ?? "")
    }
}
```

---

## Domain Layer

### Entities

```swift
// User Entity
struct User {
    let id: String
    let name: String
    let email: String
    let avatar: String?
    let role: Role
    
    enum Role {
        case admin
        case user
        case guest
    }
    
    func canEdit() -> Bool {
        role == .admin
    }
    
    func isValid() -> Bool {
        !name.isEmpty && 
        email.contains("@") && 
        email.contains(".")
    }
}

// Product Entity
struct Product {
    let id: String
    let name: String
    let description: String
    let price: Double
    let stock: Int
    let imageURL: String?
    let category: Category
    
    enum Category: String {
        case electronics
        case clothing
        case food
        case books
    }
    
    func isAvailable() -> Bool {
        stock > 0 && price > 0
    }
    
    func applyDiscount(_ percentage: Double) -> Product {
        let newPrice = price * (1 - percentage / 100)
        return Product(
            id: id,
            name: name,
            description: description,
            price: newPrice,
            stock: stock,
            imageURL: imageURL,
            category: category
        )
    }
}

// Order Entity
struct Order {
    let id: String
    let userId: String
    let items: [OrderItem]
    let status: Status
    let createdAt: Date
    
    enum Status {
        case pending
        case processing
        case shipped
        case delivered
        case cancelled
    }
    
    var total: Double {
        items.reduce(0) { $0 + $1.subtotal }
    }
    
    func canCancel() -> Bool {
        status == .pending || status == .processing
    }
}

struct OrderItem {
    let productId: String
    let quantity: Int
    let price: Double
    
    var subtotal: Double {
        Double(quantity) * price
    }
}
```

### Use Cases

```swift
// Fetch Products Use Case
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

// Create Order Use Case
protocol CreateOrderUseCase {
    func execute(userId: String, items: [OrderItem]) async throws -> Order
}

class CreateOrderUseCaseImpl: CreateOrderUseCase {
    private let orderRepository: OrderRepository
    private let productRepository: ProductRepository
    
    init(
        orderRepository: OrderRepository,
        productRepository: ProductRepository
    ) {
        self.orderRepository = orderRepository
        self.productRepository = productRepository
    }
    
    func execute(userId: String, items: [OrderItem]) async throws -> Order {
        // Validate items
        for item in items {
            let product = try await productRepository.getProduct(id: item.productId)
            
            guard product.isAvailable() else {
                throw OrderError.productNotAvailable
            }
            
            guard product.stock >= item.quantity else {
                throw OrderError.insufficientStock
            }
        }
        
        // Create order
        let order = Order(
            id: UUID().uuidString,
            userId: userId,
            items: items,
            status: .pending,
            createdAt: Date()
        )
        
        // Save order
        try await orderRepository.saveOrder(order)
        
        // Update stock
        for item in items {
            try await productRepository.updateStock(
                productId: item.productId,
                quantity: -item.quantity
            )
        }
        
        return order
    }
}

enum OrderError: Error {
    case productNotAvailable
    case insufficientStock
    case invalidQuantity
}
```

### Repository Protocols

```swift
protocol ProductRepository {
    func fetchProducts() async throws -> [Product]
    func getProduct(id: String) async throws -> Product
    func saveProduct(_ product: Product) async throws
    func deleteProduct(id: String) async throws
    func updateStock(productId: String, quantity: Int) async throws
}

protocol OrderRepository {
    func fetchOrders(userId: String) async throws -> [Order]
    func getOrder(id: String) async throws -> Order
    func saveOrder(_ order: Order) async throws
    func updateOrderStatus(id: String, status: Order.Status) async throws
}

protocol UserRepository {
    func fetchUsers() async throws -> [User]
    func getUser(id: String) async throws -> User
    func saveUser(_ user: User) async throws
    func deleteUser(id: String) async throws
    func updateUserRole(id: String, role: User.Role) async throws
}
```

---

## Data Layer

### Repository Implementation

```swift
class ProductRepositoryImpl: ProductRepository {
    private let remoteDataSource: ProductRemoteDataSource
    private let localDataSource: ProductLocalDataSource
    
    init(
        remoteDataSource: ProductRemoteDataSource,
        localDataSource: ProductLocalDataSource
    ) {
        self.remoteDataSource = remoteDataSource
        self.localDataSource = localDataSource
    }
    
    func fetchProducts() async throws -> [Product] {
        do {
            // Try remote first
            let dtos = try await remoteDataSource.fetchProducts()
            let products = dtos.map { $0.toDomain() }
            
            // Cache locally
            try await localDataSource.saveProducts(dtos)
            
            return products
        } catch {
            // Fallback to local
            let dtos = try await localDataSource.getProducts()
            return dtos.map { $0.toDomain() }
        }
    }
    
    func getProduct(id: String) async throws -> Product {
        // Try local first
        if let dto = try? await localDataSource.getProduct(id: id) {
            return dto.toDomain()
        }
        
        // Fetch from remote
        let dto = try await remoteDataSource.getProduct(id: id)
        try? await localDataSource.saveProduct(dto)
        
        return dto.toDomain()
    }
    
    func saveProduct(_ product: Product) async throws {
        let dto = ProductDTO(from: product)
        try await remoteDataSource.saveProduct(dto)
        try await localDataSource.saveProduct(dto)
    }
    
    func deleteProduct(id: String) async throws {
        try await remoteDataSource.deleteProduct(id: id)
        try await localDataSource.deleteProduct(id: id)
    }
    
    func updateStock(productId: String, quantity: Int) async throws {
        try await remoteDataSource.updateStock(productId: productId, quantity: quantity)
        try await localDataSource.updateStock(productId: productId, quantity: quantity)
    }
}
```

### Data Transfer Objects

```swift
struct ProductDTO: Codable {
    let id: String
    let name: String
    let description: String
    let price: Double
    let stock: Int
    let imageURL: String?
    let category: String
    
    func toDomain() -> Product {
        Product(
            id: id,
            name: name,
            description: description,
            price: price,
            stock: stock,
            imageURL: imageURL,
            category: Product.Category(rawValue: category) ?? .electronics
        )
    }
    
    init(from domain: Product) {
        self.id = domain.id
        self.name = domain.name
        self.description = domain.description
        self.price = domain.price
        self.stock = domain.stock
        self.imageURL = domain.imageURL
        self.category = domain.category.rawValue
    }
}

struct OrderDTO: Codable {
    let id: String
    let userId: String
    let items: [OrderItemDTO]
    let status: String
    let createdAt: String
    
    func toDomain() -> Order {
        Order(
            id: id,
            userId: userId,
            items: items.map { $0.toDomain() },
            status: Order.Status(from: status),
            createdAt: ISO8601DateFormatter().date(from: createdAt) ?? Date()
        )
    }
}

struct OrderItemDTO: Codable {
    let productId: String
    let quantity: Int
    let price: Double
    
    func toDomain() -> OrderItem {
        OrderItem(productId: productId, quantity: quantity, price: price)
    }
}
```

---

## Dependency Rule

### Correcto vs Incorrecto

```swift
// ❌ Incorrecto - Domain depende de Data
// Domain Layer
import Foundation
import CoreData // ❌ NO!

struct User {
    let entity: NSManagedObject // ❌ NO!
}

// ✅ Correcto - Data depende de Domain
// Domain Layer
struct User {
    let id: String
    let name: String
}

// Data Layer
import CoreData

extension UserEntity {
    func toDomain() -> User {
        User(id: id, name: name)
    }
}
```

---

## Use Cases

### Complex Use Case Example

```swift
protocol ProcessPaymentUseCase {
    func execute(orderId: String, paymentMethod: PaymentMethod) async throws -> Payment
}

class ProcessPaymentUseCaseImpl: ProcessPaymentUseCase {
    private let orderRepository: OrderRepository
    private let paymentGateway: PaymentGateway
    private let emailService: EmailService
    
    init(
        orderRepository: OrderRepository,
        paymentGateway: PaymentGateway,
        emailService: EmailService
    ) {
        self.orderRepository = orderRepository
        self.paymentGateway = paymentGateway
        self.emailService = emailService
    }
    
    func execute(orderId: String, paymentMethod: PaymentMethod) async throws -> Payment {
        // 1. Get order
        let order = try await orderRepository.getOrder(id: orderId)
        
        // 2. Validate order
        guard order.status == .pending else {
            throw PaymentError.invalidOrderStatus
        }
        
        // 3. Process payment
        let payment = try await paymentGateway.charge(
            amount: order.total,
            method: paymentMethod
        )
        
        // 4. Update order status
        try await orderRepository.updateOrderStatus(
            id: orderId,
            status: .processing
        )
        
        // 5. Send confirmation email
        try? await emailService.sendOrderConfirmation(order: order)
        
        return payment
    }
}
```

---

## Entities

### Rich Domain Models

```swift
struct Cart {
    private(set) var items: [CartItem] = []
    
    var total: Double {
        items.reduce(0) { $0 + $1.subtotal }
    }
    
    var isEmpty: Bool {
        items.isEmpty
    }
    
    mutating func addItem(product: Product, quantity: Int) {
        if let index = items.firstIndex(where: { $0.productId == product.id }) {
            items[index].quantity += quantity
        } else {
            items.append(CartItem(
                productId: product.id,
                name: product.name,
                price: product.price,
                quantity: quantity
            ))
        }
    }
    
    mutating func removeItem(productId: String) {
        items.removeAll { $0.productId == productId }
    }
    
    mutating func updateQuantity(productId: String, quantity: Int) {
        guard let index = items.firstIndex(where: { $0.productId == productId }) else {
            return
        }
        
        if quantity <= 0 {
            items.remove(at: index)
        } else {
            items[index].quantity = quantity
        }
    }
    
    mutating func clear() {
        items.removeAll()
    }
}

struct CartItem {
    let productId: String
    let name: String
    let price: Double
    var quantity: Int
    
    var subtotal: Double {
        price * Double(quantity)
    }
}
```

---

## Repositories

### Repository con Cache Strategy

```swift
class CachedProductRepository: ProductRepository {
    private let remote: ProductRemoteDataSource
    private let local: ProductLocalDataSource
    private let cache: Cache<String, Product>
    
    init(
        remote: ProductRemoteDataSource,
        local: ProductLocalDataSource,
        cache: Cache<String, Product> = Cache()
    ) {
        self.remote = remote
        self.local = local
        self.cache = cache
    }
    
    func fetchProducts() async throws -> [Product] {
        // 1. Check memory cache
        if let cached = cache.value(forKey: "all_products") as? [Product] {
            return cached
        }
        
        // 2. Try local database
        if let local Products = try? await local.getProducts() {
            let products = localProducts.map { $0.toDomain() }
            cache.setValue(products, forKey: "all_products")
            return products
        }
        
        // 3. Fetch from API
        let dtos = try await remote.fetchProducts()
        let products = dtos.map { $0.toDomain() }
        
        // 4. Cache results
        cache.setValue(products, forKey: "all_products")
        try? await local.saveProducts(dtos)
        
        return products
    }
}
```

---

## Casos de Uso Completos

### E-Commerce Flow Completo

```swift
// 1. Agregar producto al carrito
protocol AddToCartUseCase {
    func execute(productId: String, quantity: Int) async throws
}

// 2. Ver carrito
protocol GetCartUseCase {
    func execute() async throws -> Cart
}

// 3. Realizar pedido
protocol CheckoutUseCase {
    func execute(userId: String) async throws -> Order
}

// 4. Procesar pago
protocol ProcessPaymentUseCase {
    func execute(orderId: String, paymentMethod: PaymentMethod) async throws -> Payment
}
```

---

## Testing

### Unit Testing Use Cases

```swift
@MainActor
class CreateOrderUseCaseTests: XCTestCase {
    var useCase: CreateOrderUseCase!
    var mockOrderRepo: MockOrderRepository!
    var mockProductRepo: MockProductRepository!
    
    override func setUp() {
        mockOrderRepo = MockOrderRepository()
        mockProductRepo = MockProductRepository()
        useCase = CreateOrderUseCaseImpl(
            orderRepository: mockOrderRepo,
            productRepository: mockProductRepo
        )
    }
    
    func testCreateOrder_Success() async throws {
        // Given
        let product = Product(
            id: "1",
            name: "Test",
            description: "",
            price: 10.0,
            stock: 5,
            imageURL: nil,
            category: .electronics
        )
        mockProductRepo.products = [product]
        
        let items = [OrderItem(productId: "1", quantity: 2, price: 10.0)]
        
        // When
        let order = try await useCase.execute(userId: "user1", items: items)
        
        // Then
        XCTAssertEqual(order.items.count, 1)
        XCTAssertEqual(order.total, 20.0)
        XCTAssertTrue(mockOrderRepo.savedOrders.contains(where: { $0.id == order.id }))
    }
}
```

---

## Best Practices

### 1. Dependency Inversion

```swift
// ✅ Bueno
protocol UserRepository {
    func fetchUsers() async throws -> [User]
}

class UserRepositoryImpl: UserRepository {
    // Implementation
}

// ✅ Use Case depende de abstracción
class FetchUsersUseCase {
    private let repository: UserRepository
}
```

### 2. Single Responsibility

```swift
// ✅ Bueno - Cada clase una responsabilidad
class FetchUsersUseCase { }
class CreateUserUseCase { }
class DeleteUserUseCase { }
```

### 3. Testability

```swift
// ✅ Bueno - Fácil de testear
class MockRepository: UserRepository {
    var usersToReturn: [User] = []
    
    func fetchUsers() async throws -> [User] {
        usersToReturn
    }
}
```

---

## Checklist de Aprendizaje

### Básico
- [ ] Entender las 3 capas principales
- [ ] Crear Entities simples
- [ ] Implementar Use Cases básicos
- [ ] Separar Domain de Data

### Intermedio
- [ ] Dependency Rule
- [ ] Repository Pattern
- [ ] DTOs y mappers
- [ ] Testing de Use Cases

### Avanzado
- [ ] Complex business logic
- [ ] Cache strategies
- [ ] Error handling avanzado
- [ ] Performance optimization

---

## Próximos Pasos

### 1. Arquitecturas Complementarias
- MVI (Model-View-Intent)
- Redux/TCA
- Coordinator Pattern

### 2. Advanced Topics
- Modularización
- Dependency Injection containers
- Feature flags

---

## Recursos

### Documentación
- 📚 [Clean Architecture Book](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164)
- 📖 [Uncle Bob](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

### Videos
- 🎥 [Clean Architecture](https://www.youtube.com/watch?v=o_TH-Y78tt4)
- 🎥 [iOS Clean Architecture](https://www.youtube.com/watch?v=4GjXq2Sr55Q)

### Artículos
- 📝 [Clean Architecture in Swift](https://tech.olx.com/clean-architecture-and-mvvm-on-ios-c9d167d9f5b3)
- 📝 [Swift by Sundell](https://www.swiftbysundell.com/articles/building-an-app-using-clean-architecture/)

---

**Anterior:** [MVVM](MVVM.md)  
**Próximo:** [Repository Pattern](Repository-Pattern.md)
