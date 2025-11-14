# MVVM en iOS/SwiftUI - Guía Completa

## Tabla de Contenido
1. [¿Qué es MVVM?](#qué-es-mvvm)
2. [View Layer](#view-layer)
3. [ViewModel Layer](#viewmodel-layer)
4. [Model Layer](#model-layer)
5. [Data Binding](#data-binding)
6. [Dependency Injection](#dependency-injection)
7. [Testing ViewModels](#testing-viewmodels)
8. [Casos de Uso Reales](#casos-de-uso-reales)
9. [Best Practices](#best-practices)
10. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
11. [Próximos Pasos](#próximos-pasos)
12. [Recursos](#recursos)

---

## ¿Qué es MVVM?

**Model-View-ViewModel** es un patrón arquitectónico que separa la lógica de presentación de la lógica de negocio y la UI.

### Componentes

```
┌────────────────────────────────────┐
│           VIEW                      │
│  (SwiftUI / UIKit)                 │
│  - Presenta datos                  │
│  - Captura eventos del usuario     │
└─────────┬──────────────────────────┘
          │ Observa @Published
          ▼
┌────────────────────────────────────┐
│        VIEW MODEL                   │
│  (ObservableObject)                │
│  - Lógica de presentación          │
│  - Transformación de datos         │
│  - Estado de la UI                 │
└─────────┬──────────────────────────┘
          │ Usa
          ▼
┌────────────────────────────────────┐
│          MODEL                      │
│  - Lógica de negocio               │
│  - Modelos de datos                │
│  - Servicios/Repositories          │
└────────────────────────────────────┘
```

### Ventajas

```swift
// ✅ Ventajas de MVVM

// 1. Separación de responsabilidades
// - View: Solo UI
// - ViewModel: Lógica de presentación
// - Model: Datos y lógica de negocio

// 2. Testeable
// - ViewModels pueden testearse sin UI
// - Mocks fáciles de implementar

// 3. Reutilizable
// - ViewModels pueden compartirse
// - Lógica independiente de la UI

// 4. Mantenible
// - Código organizado
// - Fácil de modificar
```

---

## View Layer

### SwiftUI View

```swift
struct UserListView: View {
    @StateObject private var viewModel = UserListViewModel()
    
    var body: some View {
        NavigationStack {
            content
                .navigationTitle("Users")
                .toolbar {
                    ToolbarItem(placement: .navigationBarTrailing) {
                        addButton
                    }
                }
        }
        .task {
            await viewModel.loadUsers()
        }
        .alert("Error", isPresented: $viewModel.showError) {
            Button("OK") { }
        } message: {
            Text(viewModel.errorMessage ?? "Unknown error")
        }
    }
    
    @ViewBuilder
    private var content: some View {
        if viewModel.isLoading {
            ProgressView()
        } else if viewModel.users.isEmpty {
            emptyState
        } else {
            userList
        }
    }
    
    private var userList: some View {
        List(viewModel.users) { user in
            UserRow(user: user)
                .onTapGesture {
                    viewModel.selectUser(user)
                }
        }
        .refreshable {
            await viewModel.refresh()
        }
    }
    
    private var emptyState: some View {
        ContentUnavailableView(
            "No Users",
            systemImage: "person.slash",
            description: Text("Add your first user")
        )
    }
    
    private var addButton: some View {
        Button {
            viewModel.showAddUser = true
        } label: {
            Image(systemName: "plus")
        }
        .sheet(isPresented: $viewModel.showAddUser) {
            AddUserView()
        }
    }
}
```

### UIKit View (Opcional)

```swift
class UserListViewController: UIViewController {
    private var viewModel: UserListViewModel!
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var tableView: UITableView = {
        let table = UITableView()
        table.register(UserCell.self, forCellReuseIdentifier: "UserCell")
        table.dataSource = self
        table.delegate = self
        return table
    }()
    
    private lazy var activityIndicator: UIActivityIndicatorView = {
        let indicator = UIActivityIndicatorView(style: .large)
        indicator.hidesWhenStopped = true
        return indicator
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        bindViewModel()
        
        Task {
            await viewModel.loadUsers()
        }
    }
    
    private func setupUI() {
        view.addSubview(tableView)
        view.addSubview(activityIndicator)
        // Layout constraints...
    }
    
    private func bindViewModel() {
        viewModel.$users
            .receive(on: DispatchQueue.main)
            .sink { [weak self] _ in
                self?.tableView.reloadData()
            }
            .store(in: &cancellables)
        
        viewModel.$isLoading
            .receive(on: DispatchQueue.main)
            .sink { [weak self] isLoading in
                if isLoading {
                    self?.activityIndicator.startAnimating()
                } else {
                    self?.activityIndicator.stopAnimating()
                }
            }
            .store(in: &cancellables)
    }
}
```

---

## ViewModel Layer

### Basic ViewModel

```swift
@MainActor
class UserListViewModel: ObservableObject {
    // MARK: - Published Properties
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    @Published var showError = false
    @Published var showAddUser = false
    
    // MARK: - Dependencies
    private let userRepository: UserRepositoryProtocol
    
    // MARK: - Initialization
    init(userRepository: UserRepositoryProtocol = UserRepository()) {
        self.userRepository = userRepository
    }
    
    // MARK: - Actions
    func loadUsers() async {
        isLoading = true
        errorMessage = nil
        
        do {
            users = try await userRepository.fetchUsers()
        } catch {
            handleError(error)
        }
        
        isLoading = false
    }
    
    func refresh() async {
        await loadUsers()
    }
    
    func selectUser(_ user: User) {
        print("Selected user: \(user.name)")
    }
    
    func deleteUser(_ user: User) async {
        do {
            try await userRepository.deleteUser(user)
            users.removeAll { $0.id == user.id }
        } catch {
            handleError(error)
        }
    }
    
    // MARK: - Private Methods
    private func handleError(_ error: Error) {
        errorMessage = error.localizedDescription
        showError = true
    }
}
```

### Advanced ViewModel con State

```swift
@MainActor
class ProductListViewModel: ObservableObject {
    // MARK: - State
    enum State {
        case idle
        case loading
        case loaded([Product])
        case error(Error)
    }
    
    // MARK: - Published Properties
    @Published private(set) var state: State = .idle
    @Published var searchText = ""
    @Published var selectedCategory: Category?
    
    // MARK: - Computed Properties
    var filteredProducts: [Product] {
        guard case .loaded(let products) = state else { return [] }
        
        var filtered = products
        
        // Filter by search
        if !searchText.isEmpty {
            filtered = filtered.filter { $0.name.localizedCaseInsensitiveContains(searchText) }
        }
        
        // Filter by category
        if let category = selectedCategory {
            filtered = filtered.filter { $0.category == category }
        }
        
        return filtered
    }
    
    var isLoading: Bool {
        if case .loading = state {
            return true
        }
        return false
    }
    
    var errorMessage: String? {
        if case .error(let error) = state {
            return error.localizedDescription
        }
        return nil
    }
    
    // MARK: - Dependencies
    private let repository: ProductRepository
    private var searchTask: Task<Void, Never>?
    
    // MARK: - Initialization
    init(repository: ProductRepository = ProductRepository()) {
        self.repository = repository
        setupSearchDebounce()
    }
    
    // MARK: - Actions
    func loadProducts() async {
        state = .loading
        
        do {
            let products = try await repository.fetchProducts()
            state = .loaded(products)
        } catch {
            state = .error(error)
        }
    }
    
    func retry() async {
        await loadProducts()
    }
    
    // MARK: - Private Methods
    private func setupSearchDebounce() {
        $searchText
            .debounce(for: .milliseconds(300), scheduler: DispatchQueue.main)
            .sink { [weak self] _ in
                self?.objectWillChange.send()
            }
    }
}
```

---

## Model Layer

### Domain Models

```swift
// MARK: - User Model
struct User: Identifiable, Codable {
    let id: Int
    let name: String
    let email: String
    let avatar: String?
    
    var initials: String {
        let components = name.components(separatedBy: " ")
        return components.compactMap { $0.first }.map { String($0) }.joined()
    }
}

// MARK: - Product Model
struct Product: Identifiable, Codable {
    let id: Int
    let name: String
    let price: Double
    let category: Category
    let imageURL: String?
    
    var formattedPrice: String {
        "$\(String(format: "%.2f", price))"
    }
}

enum Category: String, Codable, CaseIterable {
    case electronics = "Electronics"
    case clothing = "Clothing"
    case food = "Food"
    case books = "Books"
}
```

### Repository

```swift
protocol UserRepositoryProtocol {
    func fetchUsers() async throws -> [User]
    func getUser(id: Int) async throws -> User
    func createUser(_ user: User) async throws -> User
    func updateUser(_ user: User) async throws -> User
    func deleteUser(_ user: User) async throws
}

class UserRepository: UserRepositoryProtocol {
    private let apiClient: APIClient
    private let cacheManager: CacheManager
    
    init(
        apiClient: APIClient = APIClient.shared,
        cacheManager: CacheManager = CacheManager.shared
    ) {
        self.apiClient = apiClient
        self.cacheManager = cacheManager
    }
    
    func fetchUsers() async throws -> [User] {
        // Try cache first
        if let cached: [User] = cacheManager.get(key: "users") {
            return cached
        }
        
        // Fetch from API
        let users: [User] = try await apiClient.request("users")
        
        // Cache result
        cacheManager.set(users, key: "users")
        
        return users
    }
    
    func getUser(id: Int) async throws -> User {
        try await apiClient.request("users/\(id)")
    }
    
    func createUser(_ user: User) async throws -> User {
        try await apiClient.post("users", body: user)
    }
    
    func updateUser(_ user: User) async throws -> User {
        try await apiClient.put("users/\(user.id)", body: user)
    }
    
    func deleteUser(_ user: User) async throws {
        try await apiClient.delete("users/\(user.id)")
        cacheManager.remove(key: "users")
    }
}
```

---

## Data Binding

### One-Way Binding

```swift
@MainActor
class CounterViewModel: ObservableObject {
    @Published private(set) var count = 0
    
    func increment() {
        count += 1
    }
    
    func decrement() {
        count -= 1
    }
}

struct CounterView: View {
    @StateObject private var viewModel = CounterViewModel()
    
    var body: some View {
        VStack {
            Text("\(viewModel.count)")
                .font(.largeTitle)
            
            HStack {
                Button("−") {
                    viewModel.decrement()
                }
                
                Button("+") {
                    viewModel.increment()
                }
            }
        }
    }
}
```

### Two-Way Binding

```swift
@MainActor
class FormViewModel: ObservableObject {
    @Published var name = ""
    @Published var email = ""
    @Published var age = 18
    
    var isValid: Bool {
        !name.isEmpty && email.contains("@") && age >= 18
    }
    
    func submit() {
        print("Submitting: \(name), \(email), \(age)")
    }
}

struct FormView: View {
    @StateObject private var viewModel = FormViewModel()
    
    var body: some View {
        Form {
            TextField("Name", text: $viewModel.name)
            TextField("Email", text: $viewModel.email)
            Stepper("Age: \(viewModel.age)", value: $viewModel.age)
            
            Button("Submit") {
                viewModel.submit()
            }
            .disabled(!viewModel.isValid)
        }
    }
}
```

---

## Dependency Injection

### Constructor Injection

```swift
@MainActor
class UserDetailViewModel: ObservableObject {
    @Published var user: User?
    @Published var isLoading = false
    
    private let userID: Int
    private let repository: UserRepositoryProtocol
    
    init(userID: Int, repository: UserRepositoryProtocol = UserRepository()) {
        self.userID = userID
        self.repository = repository
    }
    
    func loadUser() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            user = try await repository.getUser(id: userID)
        } catch {
            print("Error: \(error)")
        }
    }
}
```

### Environment Injection

```swift
class AppDependencies {
    static let shared = AppDependencies()
    
    lazy var userRepository: UserRepositoryProtocol = {
        UserRepository()
    }()
    
    lazy var productRepository: ProductRepository = {
        ProductRepository()
    }()
    
    func makeUserListViewModel() -> UserListViewModel {
        UserListViewModel(userRepository: userRepository)
    }
    
    func makeProductListViewModel() -> ProductListViewModel {
        ProductListViewModel(repository: productRepository)
    }
}

// Usage in App
@main
struct MyApp: App {
    let dependencies = AppDependencies.shared
    
    var body: some Scene {
        WindowGroup {
            UserListView()
                .environmentObject(dependencies.makeUserListViewModel())
        }
    }
}
```

---

## Testing ViewModels

### Basic Unit Test

```swift
import XCTest
@testable import MyApp

@MainActor
class UserListViewModelTests: XCTestCase {
    var viewModel: UserListViewModel!
    var mockRepository: MockUserRepository!
    
    override func setUp() async throws {
        mockRepository = MockUserRepository()
        viewModel = UserListViewModel(userRepository: mockRepository)
    }
    
    override func tearDown() {
        viewModel = nil
        mockRepository = nil
    }
    
    func testLoadUsers_Success() async throws {
        // Given
        let expectedUsers = [
            User(id: 1, name: "Test User", email: "test@example.com", avatar: nil)
        ]
        mockRepository.usersToReturn = expectedUsers
        
        // When
        await viewModel.loadUsers()
        
        // Then
        XCTAssertEqual(viewModel.users.count, 1)
        XCTAssertEqual(viewModel.users.first?.name, "Test User")
        XCTAssertFalse(viewModel.isLoading)
        XCTAssertNil(viewModel.errorMessage)
    }
    
    func testLoadUsers_Failure() async throws {
        // Given
        mockRepository.shouldFail = true
        
        // When
        await viewModel.loadUsers()
        
        // Then
        XCTAssertTrue(viewModel.users.isEmpty)
        XCTAssertNotNil(viewModel.errorMessage)
        XCTAssertTrue(viewModel.showError)
    }
    
    func testDeleteUser() async throws {
        // Given
        let user = User(id: 1, name: "Test", email: "test@example.com", avatar: nil)
        viewModel.users = [user]
        
        // When
        await viewModel.deleteUser(user)
        
        // Then
        XCTAssertTrue(viewModel.users.isEmpty)
    }
}

// MARK: - Mock Repository
class MockUserRepository: UserRepositoryProtocol {
    var usersToReturn: [User] = []
    var shouldFail = false
    
    func fetchUsers() async throws -> [User] {
        if shouldFail {
            throw NSError(domain: "Test", code: -1)
        }
        return usersToReturn
    }
    
    func getUser(id: Int) async throws -> User {
        User(id: id, name: "Mock User", email: "mock@example.com", avatar: nil)
    }
    
    func createUser(_ user: User) async throws -> User {
        user
    }
    
    func updateUser(_ user: User) async throws -> User {
        user
    }
    
    func deleteUser(_ user: User) async throws {
        // Do nothing
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
    
    init(authService: AuthService = AuthService.shared) {
        self.authService = authService
    }
    
    var isValidForm: Bool {
        !email.isEmpty && 
        email.contains("@") && 
        password.count >= 6
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
}

struct LoginView: View {
    @StateObject private var viewModel = LoginViewModel()
    
    var body: some View {
        VStack(spacing: 20) {
            TextField("Email", text: $viewModel.email)
                .textFieldStyle(.roundedBorder)
                .textContentType(.emailAddress)
                .autocapitalization(.none)
            
            SecureField("Password", text: $viewModel.password)
                .textFieldStyle(.roundedBorder)
            
            if let error = viewModel.errorMessage {
                Text(error)
                    .foregroundColor(.red)
                    .font(.caption)
            }
            
            Button("Login") {
                Task {
                    await viewModel.login()
                }
            }
            .buttonStyle(.borderedProminent)
            .disabled(!viewModel.isValidForm || viewModel.isLoading)
        }
        .padding()
        .fullScreenCover(isPresented: $viewModel.isLoggedIn) {
            MainAppView()
        }
    }
}
```

### Shopping Cart

```swift
@MainActor
class CartViewModel: ObservableObject {
    @Published var items: [CartItem] = []
    @Published var isLoading = false
    
    var total: Double {
        items.reduce(0) { $0 + ($1.product.price * Double($1.quantity)) }
    }
    
    var itemCount: Int {
        items.reduce(0) { $0 + $1.quantity }
    }
    
    func addItem(_ product: Product) {
        if let index = items.firstIndex(where: { $0.product.id == product.id }) {
            items[index].quantity += 1
        } else {
            items.append(CartItem(product: product, quantity: 1))
        }
    }
    
    func removeItem(_ product: Product) {
        items.removeAll { $0.product.id == product.id }
    }
    
    func updateQuantity(for product: Product, quantity: Int) {
        guard let index = items.firstIndex(where: { $0.product.id == product.id }) else { return }
        
        if quantity <= 0 {
            items.remove(at: index)
        } else {
            items[index].quantity = quantity
        }
    }
    
    func checkout() async {
        isLoading = true
        // Checkout logic
        isLoading = false
    }
}

struct CartItem: Identifiable {
    let id = UUID()
    let product: Product
    var quantity: Int
}
```

---

## Best Practices

### 1. Single Responsibility

```swift
// ❌ Malo - ViewModel hace demasiado
class BadViewModel: ObservableObject {
    func fetchData() { }
    func saveData() { }
    func validateEmail() { }
    func formatDate() { }
    func calculateTotal() { }
}

// ✅ Bueno - Responsabilidades separadas
class UserViewModel: ObservableObject {
    private let validator: Validator
    private let formatter: Formatter
    
    func fetchUsers() async { }
    func saveUser() async { }
}
```

### 2. Immutable Models

```swift
// ✅ Bueno - Immutable struct
struct User: Identifiable {
    let id: Int
    let name: String
    let email: String
}
```

### 3. Dependency Injection

```swift
// ✅ Bueno - Inyección de dependencias
@MainActor
class ViewModel: ObservableObject {
    private let repository: Repository
    
    init(repository: Repository = Repository()) {
        self.repository = repository
    }
}
```

---

## Checklist de Aprendizaje

### Básico
- [ ] Entender MVVM pattern
- [ ] Crear ViewModels con @Published
- [ ] Separar View, ViewModel, Model
- [ ] Usar @StateObject y @ObservedObject

### Intermedio
- [ ] Dependency injection
- [ ] Testing ViewModels
- [ ] Error handling robusto
- [ ] State management patterns

### Avanzado
- [ ] Complex state machines
- [ ] Performance optimization
- [ ] Memory management
- [ ] Advanced testing strategies

---

## Próximos Pasos

### 1. Clean Architecture
- Capas bien definidas
- Use Cases
- Repository Pattern

### 2. Advanced Patterns
- Coordinator Pattern
- Redux/TCA
- Unidirectional Data Flow

---

## Recursos

### Documentación
- 📚 [MVVM Pattern](https://developer.apple.com/documentation/swiftui/model-data)
- 📖 [ObservableObject](https://developer.apple.com/documentation/combine/observableobject)

### Videos
- 🎥 [MVVM in SwiftUI](https://www.youtube.com/watch?v=4GjXq2Sr55Q)
- 🎥 [Advanced MVVM](https://developer.apple.com/videos/play/wwdc2019/226/)

### Artículos
- 📝 [Hacking with Swift - MVVM](https://www.hackingwithswift.com/books/ios-swiftui/introducing-mvvm-into-your-swiftui-project)
- 📝 [Swift by Sundell](https://www.swiftbysundell.com/articles/different-flavors-of-dependency-injection-in-swift/)

---

**Anterior:** [Animations](../02-SwiftUI/Animations.md)  
**Próximo:** [Clean Architecture](Clean-Architecture.md)
