# Actors en Swift - Guía Completa

## Tabla de Contenido
1. [¿Qué son los Actors?](#qué-son-los-actors)
2. [Problema de Race Conditions](#problema-de-race-conditions)
3. [Actor Básico](#actor-básico)
4. [Actor Isolation](#actor-isolation)
5. [MainActor](#mainactor)
6. [Global Actors](#global-actors)
7. [Nonisolated](#nonisolated)
8. [Actor Reentrancy](#actor-reentrancy)
9. [Sendable Protocol](#sendable-protocol)
10. [Casos de Uso Reales](#casos-de-uso-reales)
11. [Best Practices](#best-practices)
12. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
13. [Próximos Pasos](#próximos-pasos)
14. [Recursos](#recursos)

---

## ¿Qué son los Actors?

Los **Actors** son un tipo de referencia introducido en Swift 5.5 que protege automáticamente su estado mutable del acceso concurrente, eliminando las race conditions.

### Concepto

```swift
// ✅ Actor protege su estado automáticamente
actor BankAccount {
    private var balance = 0.0
    
    func deposit(_ amount: Double) {
        balance += amount  // ✅ Thread-safe automáticamente
    }
    
    func withdraw(_ amount: Double) {
        balance -= amount  // ✅ Thread-safe automáticamente
    }
}

// ❌ Class sin protección
class UnsafeBankAccount {
    private var balance = 0.0
    
    func deposit(_ amount: Double) {
        balance += amount  // ⚠️ Race condition!
    }
}
```

### Características Principales

```swift
// 1. Reference Type
// - Como classes
// - Pero con protección de concurrencia

// 2. Isolation
// - El estado está aislado
// - Solo una tarea puede acceder a la vez

// 3. Async Access
// - Acceso desde fuera requiere await
// - Sincronización automática

// 4. Thread-Safe
// - No necesitas DispatchQueue
// - No necesitas locks/semáforos
```

---

## Problema de Race Conditions

### Sin Actors (Problemático)

```swift
// ❌ PROBLEMA: Race Condition
class Counter {
    private var value = 0
    
    func increment() {
        value += 1
    }
    
    func getValue() -> Int {
        value
    }
}

// Múltiples threads accediendo simultáneamente
let counter = Counter()

// Thread 1
Task {
    for _ in 0..<1000 {
        counter.increment()
    }
}

// Thread 2
Task {
    for _ in 0..<1000 {
        counter.increment()
    }
}

// Resultado: ⚠️ Valor impredecible (no será 2000)
```

### Con Actors (Seguro)

```swift
// ✅ SOLUCIÓN: Actor
actor Counter {
    private var value = 0
    
    func increment() {
        value += 1
    }
    
    func getValue() -> Int {
        value
    }
}

// Uso seguro
let counter = Counter()

// Thread 1
Task {
    for _ in 0..<1000 {
        await counter.increment()
    }
}

// Thread 2
Task {
    for _ in 0..<1000 {
        await counter.increment()
    }
}

// Resultado: ✅ Siempre 2000
```

---

## Actor Básico

### Definición Simple

```swift
actor TemperatureLogger {
    private var measurements: [Double] = []
    
    func addMeasurement(_ temp: Double) {
        measurements.append(temp)
    }
    
    func getAverage() -> Double {
        guard !measurements.isEmpty else { return 0 }
        return measurements.reduce(0, +) / Double(measurements.count)
    }
    
    func getMeasurementCount() -> Int {
        measurements.count
    }
}

// Uso
let logger = TemperatureLogger()

Task {
    await logger.addMeasurement(25.5)
    await logger.addMeasurement(26.0)
    await logger.addMeasurement(24.8)
    
    let average = await logger.getAverage()
    print("Average: \(average)")
}
```

### Actor con Properties

```swift
actor ImageCache {
    private var cache: [String: UIImage] = [:]
    
    func getImage(for key: String) -> UIImage? {
        cache[key]
    }
    
    func setImage(_ image: UIImage, for key: String) {
        cache[key] = image
    }
    
    func removeImage(for key: String) {
        cache.removeValue(forKey: key)
    }
    
    func clearCache() {
        cache.removeAll()
    }
    
    var cacheSize: Int {
        cache.count
    }
}

// Uso
let cache = ImageCache()

Task {
    let image = UIImage(systemName: "star")!
    await cache.setImage(image, for: "star")
    
    if let cached = await cache.getImage(for: "star") {
        print("Image found in cache")
    }
    
    let size = await cache.cacheSize
    print("Cache size: \(size)")
}
```

### Actor con Métodos Async

```swift
actor DataManager {
    private var data: [String] = []
    
    func fetchData() async throws -> [String] {
        // Simula network request
        try await Task.sleep(nanoseconds: 1_000_000_000)
        return ["Data 1", "Data 2", "Data 3"]
    }
    
    func loadData() async throws {
        data = try await fetchData()
    }
    
    func getData() -> [String] {
        data
    }
}

// Uso
let manager = DataManager()

Task {
    try await manager.loadData()
    let data = await manager.getData()
    print("Loaded: \(data)")
}
```

---

## Actor Isolation

### Acceso Interno vs Externo

```swift
actor BankAccount {
    private var balance: Double = 0
    
    // ✅ Acceso interno: No requiere await
    func deposit(_ amount: Double) {
        balance += amount  // Sincrónico dentro del actor
        logTransaction("Deposit: \(amount)")
    }
    
    func withdraw(_ amount: Double) -> Bool {
        guard balance >= amount else { return false }
        balance -= amount  // Sincrónico dentro del actor
        logTransaction("Withdraw: \(amount)")
        return true
    }
    
    private func logTransaction(_ message: String) {
        print("\(message). New balance: \(balance)")
    }
    
    func getBalance() -> Double {
        balance  // Sincrónico dentro del actor
    }
}

// ⚠️ Acceso externo: Requiere await
let account = BankAccount()

Task {
    await account.deposit(100)  // await requerido
    let balance = await account.getBalance()  // await requerido
    print("Balance: \(balance)")
}
```

### Computed Properties

```swift
actor UserSession {
    private var username: String?
    private var loginTime: Date?
    
    // ✅ Property accesible desde fuera
    var isLoggedIn: Bool {
        username != nil
    }
    
    // ✅ Property con lógica
    var sessionDuration: TimeInterval? {
        guard let loginTime = loginTime else { return nil }
        return Date().timeIntervalSince(loginTime)
    }
    
    func login(username: String) {
        self.username = username
        self.loginTime = Date()
    }
    
    func logout() {
        username = nil
        loginTime = nil
    }
}

// Uso
let session = UserSession()

Task {
    await session.login(username: "user123")
    
    let isLoggedIn = await session.isLoggedIn
    print("Logged in: \(isLoggedIn)")
    
    if let duration = await session.sessionDuration {
        print("Session duration: \(duration)")
    }
}
```

---

## MainActor

### UI Updates

```swift
// ✅ MainActor garantiza ejecución en main thread
@MainActor
class ViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var error: String?
    
    private let repository: UserRepository
    
    init(repository: UserRepository) {
        self.repository = repository
    }
    
    func loadUsers() async {
        isLoading = true
        
        do {
            users = try await repository.fetchUsers()
        } catch {
            self.error = error.localizedDescription
        }
        
        isLoading = false
    }
}

// Uso en SwiftUI
struct UserListView: View {
    @StateObject private var viewModel = ViewModel(repository: UserRepository())
    
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

### MainActor Functions

```swift
class DataProcessor {
    // ✅ Función específica en main thread
    @MainActor
    func updateUI(with data: [String]) {
        // Este código corre en main thread
        print("Updating UI with: \(data)")
    }
    
    // Regular function
    func processData() async {
        let data = ["Item 1", "Item 2"]
        
        // Cambio a main thread para UI
        await updateUI(with: data)
    }
}
```

### MainActor Properties

```swift
class ImageLoader {
    @MainActor
    var loadedImage: UIImage?
    
    func loadImage(from url: URL) async {
        // Background work
        let (data, _) = try? await URLSession.shared.data(from: url)
        let image = data.flatMap { UIImage(data: $0) }
        
        // Update UI property on main thread
        await MainActor.run {
            self.loadedImage = image
        }
    }
}
```

---

## Global Actors

### Custom Global Actor

```swift
@globalActor
actor DatabaseActor {
    static let shared = DatabaseActor()
}

// Uso del global actor
@DatabaseActor
class DatabaseManager {
    private var connection: DatabaseConnection?
    
    func connect() {
        // Todas las llamadas a esta clase
        // se serializan a través de DatabaseActor
    }
    
    func query(_ sql: String) -> [Row] {
        // Thread-safe automáticamente
        []
    }
}

// Métodos aislados al global actor
@DatabaseActor
func performDatabaseOperation() {
    let manager = DatabaseManager()
    manager.connect()
}
```

### NetworkActor Example

```swift
@globalActor
actor NetworkActor {
    static let shared = NetworkActor()
}

@NetworkActor
class APIClient {
    private var session: URLSession
    
    init() {
        self.session = URLSession.shared
    }
    
    func request<T: Decodable>(_ endpoint: String) async throws -> T {
        let url = URL(string: "https://api.example.com/\(endpoint)")!
        let (data, _) = try await session.data(from: url)
        return try JSONDecoder().decode(T.self, from: data)
    }
}
```

---

## Nonisolated

### Nonisolated Methods

```swift
actor Logger {
    private var logs: [String] = []
    
    func log(_ message: String) {
        logs.append(message)
    }
    
    // ✅ No requiere await, no accede a estado mutable
    nonisolated func formatMessage(_ message: String) -> String {
        return "[\(Date())] \(message)"
    }
    
    // ✅ Puede llamar a nonisolated
    func logFormatted(_ message: String) {
        let formatted = formatMessage(message)
        logs.append(formatted)
    }
}

// Uso
let logger = Logger()

// No requiere await
let formatted = logger.formatMessage("Hello")

// Requiere await
Task {
    await logger.log("Message")
}
```

### Nonisolated Properties

```swift
actor Configuration {
    private var settings: [String: Any] = [:]
    
    // ✅ Constante, puede ser nonisolated
    nonisolated let appName = "MyApp"
    nonisolated let version = "1.0.0"
    
    func getSetting(_ key: String) -> Any? {
        settings[key]
    }
}

// Uso
let config = Configuration()

// No requiere await
let name = config.appName
let version = config.version
```

---

## Actor Reentrancy

### Problema de Reentrancy

```swift
actor DataCache {
    private var cache: [String: Data] = [:]
    
    func getData(for key: String) async -> Data? {
        // 1. Check cache
        if let cached = cache[key] {
            return cached
        }
        
        // 2. Fetch data (suspension point)
        let data = await fetchData(for: key)
        
        // ⚠️ CUIDADO: El estado puede haber cambiado!
        // Otro Task pudo haber agregado datos mientras esperábamos
        
        // 3. Cache and return
        cache[key] = data
        return data
    }
    
    private func fetchData(for key: String) async -> Data {
        // Simula network request
        Data()
    }
}
```

### Solución

```swift
actor DataCache {
    private var cache: [String: Data] = [:]
    private var inProgress: Set<String> = []
    
    func getData(for key: String) async -> Data? {
        // Check cache
        if let cached = cache[key] {
            return cached
        }
        
        // Check if already fetching
        if inProgress.contains(key) {
            // Wait and retry
            try? await Task.sleep(nanoseconds: 100_000_000)
            return await getData(for: key)
        }
        
        // Mark as in progress
        inProgress.insert(key)
        
        // Fetch data
        let data = await fetchData(for: key)
        
        // Update cache and remove from in progress
        cache[key] = data
        inProgress.remove(key)
        
        return data
    }
    
    private func fetchData(for key: String) async -> Data {
        Data()
    }
}
```

---

## Sendable Protocol

### Sendable Types

```swift
// ✅ Tipos value son Sendable
struct User: Sendable {
    let id: String
    let name: String
}

// ✅ Actor es Sendable
actor UserManager {
    private var users: [User] = []
}

// ✅ Class inmutable puede ser Sendable
final class Configuration: Sendable {
    let apiKey: String
    let baseURL: URL
    
    init(apiKey: String, baseURL: URL) {
        self.apiKey = apiKey
        self.baseURL = baseURL
    }
}

// ❌ Class mutable NO es Sendable
class Counter {  // ⚠️ No Sendable
    var value = 0
}
```

### @Sendable Closures

```swift
actor TaskManager {
    private var tasks: [() async -> Void] = []
    
    func addTask(_ task: @Sendable @escaping () async -> Void) {
        tasks.append(task)
    }
    
    func executeTasks() async {
        for task in tasks {
            await task()
        }
    }
}

// Uso
let manager = TaskManager()

await manager.addTask {
    print("Task 1")
}

await manager.addTask {
    print("Task 2")
}

await manager.executeTasks()
```

---

## Casos de Uso Reales

### Image Downloader

```swift
actor ImageDownloader {
    private var cache: [URL: UIImage] = [:]
    private var inProgress: [URL: Task<UIImage?, Never>] = [:]
    
    func downloadImage(from url: URL) async -> UIImage? {
        // Check cache
        if let cached = cache[url] {
            return cached
        }
        
        // Check if already downloading
        if let existingTask = inProgress[url] {
            return await existingTask.value
        }
        
        // Create download task
        let task = Task<UIImage?, Never> {
            do {
                let (data, _) = try await URLSession.shared.data(from: url)
                return UIImage(data: data)
            } catch {
                return nil
            }
        }
        
        inProgress[url] = task
        
        let image = await task.value
        
        // Cache result
        if let image = image {
            cache[url] = image
        }
        
        inProgress.removeValue(forKey: url)
        
        return image
    }
    
    func clearCache() {
        cache.removeAll()
    }
}
```

### Analytics Tracker

```swift
actor AnalyticsTracker {
    private var events: [(name: String, properties: [String: Any], timestamp: Date)] = []
    private let maxQueueSize = 100
    
    func track(event: String, properties: [String: Any] = [:]) {
        let entry = (name: event, properties: properties, timestamp: Date())
        events.append(entry)
        
        if events.count >= maxQueueSize {
            Task {
                await flush()
            }
        }
    }
    
    func flush() async {
        guard !events.isEmpty else { return }
        
        let eventsToSend = events
        events.removeAll()
        
        // Send to server
        await sendEvents(eventsToSend)
    }
    
    private func sendEvents(_ events: [(name: String, properties: [String: Any], timestamp: Date)]) async {
        // Implementation
        print("Sending \(events.count) events")
    }
}
```

### Database Manager

```swift
actor DatabaseManager {
    private var connection: DatabaseConnection?
    private var isConnected = false
    
    func connect() async throws {
        guard !isConnected else { return }
        
        connection = try await DatabaseConnection.create()
        isConnected = true
    }
    
    func disconnect() async {
        connection?.close()
        connection = nil
        isConnected = false
    }
    
    func query<T: Decodable>(_ sql: String) async throws -> [T] {
        guard isConnected, let connection = connection else {
            throw DatabaseError.notConnected
        }
        
        return try await connection.execute(sql)
    }
    
    func insert<T: Encodable>(_ item: T, into table: String) async throws {
        guard isConnected, let connection = connection else {
            throw DatabaseError.notConnected
        }
        
        try await connection.insert(item, into: table)
    }
}

enum DatabaseError: Error {
    case notConnected
    case queryFailed
}
```

---

## Best Practices

### 1. Prefer Actors over Locks

```swift
// ❌ Malo - Locks manuales
class DataStore {
    private var data: [String] = []
    private let lock = NSLock()
    
    func add(_ item: String) {
        lock.lock()
        data.append(item)
        lock.unlock()
    }
}

// ✅ Bueno - Actor
actor DataStore {
    private var data: [String] = []
    
    func add(_ item: String) {
        data.append(item)
    }
}
```

### 2. MainActor para UI

```swift
// ✅ Bueno - MainActor para ViewModels
@MainActor
class ViewModel: ObservableObject {
    @Published var state: State = .idle
    
    func loadData() async {
        // Updates son thread-safe en main
    }
}
```

### 3. Nonisolated cuando sea posible

```swift
actor Calculator {
    // ✅ Nonisolated para métodos puros
    nonisolated func add(_ a: Int, _ b: Int) -> Int {
        a + b
    }
}
```

---

## Checklist de Aprendizaje

### Básico
- [ ] Entender qué son los Actors
- [ ] Crear Actors básicos
- [ ] Usar await para acceder a Actors
- [ ] Entender MainActor

### Intermedio
- [ ] Actor isolation
- [ ] Nonisolated methods
- [ ] Global actors custom
- [ ] Sendable protocol

### Avanzado
- [ ] Actor reentrancy
- [ ] Complex actor patterns
- [ ] Performance optimization
- [ ] Testing con actors

---

## Próximos Pasos

### 1. Structured Concurrency
- Task groups
- Async let
- Cancellation

### 2. Advanced Patterns
- Actor coordinator
- Actor-based architecture

---

## Recursos

### Documentación
- 📚 [Actors](https://docs.swift.org/swift-book/LanguageGuide/Concurrency.html)
- 📖 [Swift Concurrency](https://developer.apple.com/documentation/swift/swift_standard_library/concurrency)

### Videos
- 🎥 [WWDC: Meet async/await](https://developer.apple.com/videos/play/wwdc2021/10132/)
- 🎥 [WWDC: Protect mutable state with Swift actors](https://developer.apple.com/videos/play/wwdc2021/10133/)

### Artículos
- 📝 [Swift Actors](https://www.swiftbysundell.com/articles/swift-actors/)
- 📝 [Actor Isolation](https://www.hackingwithswift.com/swift/5.5/actors)

---

**Anterior:** [Use Cases](../04-Arquitectura/Use-Cases.md)  
**Próximo:** [Publishers & Subscribers](../06-Combine/Publishers-Subscribers.md)
