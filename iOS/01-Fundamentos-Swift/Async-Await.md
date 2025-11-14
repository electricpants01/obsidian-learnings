# Async/Await - Concurrencia Moderna en Swift

## Tabla de Contenido
1. [Introducción a Async/Await](#introducción-a-asyncawait)
2. [Funciones Asíncronas](#funciones-asíncronas)
3. [Await Keyword](#await-keyword)
4. [Task y TaskGroup](#task-y-taskgroup)
5. [Async Let](#async-let)
6. [Actors y @MainActor](#actors-y-mainactor)
7. [Async Sequences](#async-sequences)
8. [Error Handling](#error-handling)
9. [Migración de Callbacks](#migración-de-callbacks)
10. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
11. [Próximos Pasos](#próximos-pasos)
12. [Recursos](#recursos)

---

## Introducción a Async/Await

**Async/Await** es el modelo de concurrencia moderno de Swift introducido en Swift 5.5, diseñado para reemplazar callbacks y mejorar la legibilidad del código asíncrono.

### Antes: Callbacks (Callback Hell)

```swift
// ❌ Callback Hell
func fetchUserData(completion: @escaping (User?, Error?) -> Void) {
    fetchUserID { userID, error in
        guard let userID = userID, error == nil else {
            completion(nil, error)
            return
        }
        
        fetchUserProfile(userID: userID) { profile, error in
            guard let profile = profile, error == nil else {
                completion(nil, error)
                return
            }
            
            fetchUserPosts(userID: userID) { posts, error in
                guard let posts = posts, error == nil else {
                    completion(nil, error)
                    return
                }
                
                let user = User(id: userID, profile: profile, posts: posts)
                completion(user, nil)
            }
        }
    }
}
```

### Después: Async/Await (Limpio y Legible)

```swift
// ✅ Async/Await
func fetchUserData() async throws -> User {
    let userID = try await fetchUserID()
    let profile = try await fetchUserProfile(userID: userID)
    let posts = try await fetchUserPosts(userID: userID)
    
    return User(id: userID, profile: profile, posts: posts)
}
```

---

## Funciones Asíncronas

### Declaración con `async`

```swift
// Función asíncrona simple
func fetchData() async -> Data {
    // Simular operación asíncrona
    await Task.sleep(1_000_000_000) // 1 segundo
    return Data()
}

// Con throws
func fetchDataOrThrow() async throws -> Data {
    let url = URL(string: "https://api.example.com/data")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return data
}

// Con parámetros
func fetchUser(id: String) async throws -> User {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}
```

### Propiedades Asíncronas

```swift
struct DataProvider {
    var data: Data {
        get async {
            await fetchData()
        }
    }
    
    var dataOrThrow: Data {
        get async throws {
            try await fetchDataOrThrow()
        }
    }
}

// Uso
Task {
    let provider = DataProvider()
    let data = await provider.data
}
```

---

## Await Keyword

El keyword `await` marca un punto de suspensión donde la función puede ceder control.

### Suspension Points

```swift
func processData() async {
    print("1. Start") // Ejecuta inmediatamente
    
    let data = await fetchData() // ⏸️ Suspension point
    print("2. Data fetched") // Ejecuta cuando fetchData termina
    
    let processed = await processData(data) // ⏸️ Suspension point
    print("3. Data processed")
}
```

### Await en Loops

```swift
func fetchMultipleUsers(ids: [String]) async throws -> [User] {
    var users: [User] = []
    
    for id in ids {
        let user = try await fetchUser(id: id) // Secuencial
        users.append(user)
    }
    
    return users
}
```

### Await con Optional Chaining

```swift
class UserService {
    func fetchUser() async -> User? {
        // ...
    }
}

let service: UserService? = UserService()

Task {
    // await funciona con optional chaining
    if let user = await service?.fetchUser() {
        print("User: \(user)")
    }
}
```

---

## Task y TaskGroup

### Task - Operaciones Asíncronas

```swift
// Crear una task
Task {
    let data = try await fetchData()
    print("Data: \(data)")
}

// Task con priority
Task(priority: .high) {
    await performImportantWork()
}

// Task con valor de retorno
let task = Task { () -> String in
    try await fetchUserName()
}

// Obtener resultado
let name = try await task.value
```

### Task Cancellation

```swift
let task = Task {
    for i in 1...10 {
        // Verificar cancelación
        try Task.checkCancellation()
        
        // o
        if Task.isCancelled {
            print("Task cancelled")
            return
        }
        
        await doWork(step: i)
    }
}

// Cancelar desde afuera
task.cancel()
```

### TaskGroup - Concurrencia Estructurada

```swift
func fetchAllUsers(ids: [String]) async throws -> [User] {
    try await withThrowingTaskGroup(of: User.self) { group in
        // Agregar tasks al grupo
        for id in ids {
            group.addTask {
                try await fetchUser(id: id)
            }
        }
        
        // Recolectar resultados
        var users: [User] = []
        for try await user in group {
            users.append(user)
        }
        
        return users
    }
}
```

### Limit Concurrency

```swift
func downloadImages(urls: [URL]) async throws -> [UIImage] {
    let maxConcurrent = 5
    
    return try await withThrowingTaskGroup(of: (Int, UIImage).self) { group in
        var images = [UIImage?](repeating: nil, count: urls.count)
        
        for (index, url) in urls.enumerated() {
            // Limitar concurrencia
            if index >= maxConcurrent {
                if let result = try await group.next() {
                    images[result.0] = result.1
                }
            }
            
            group.addTask {
                let (data, _) = try await URLSession.shared.data(from: url)
                guard let image = UIImage(data: data) else {
                    throw ImageError.invalidData
                }
                return (index, image)
            }
        }
        
        // Recolectar restantes
        for try await (index, image) in group {
            images[index] = image
        }
        
        return images.compactMap { $0 }
    }
}
```

---

## Async Let

Ejecutar múltiples operaciones async en paralelo:

### Parallel Execution

```swift
func fetchUserData(id: String) async throws -> (User, [Post], [Friend]) {
    // ✅ Paralelo con async let
    async let user = fetchUser(id: id)
    async let posts = fetchPosts(userId: id)
    async let friends = fetchFriends(userId: id)
    
    // Await all results
    return try await (user, posts, friends)
}

// vs Secuencial (más lento)
func fetchUserDataSequential(id: String) async throws -> (User, [Post], [Friend]) {
    let user = try await fetchUser(id: id)      // Espera
    let posts = try await fetchPosts(userId: id) // Espera
    let friends = try await fetchFriends(userId: id) // Espera
    
    return (user, posts, friends)
}
```

### Async Let con Condicionales

```swift
func loadData() async {
    async let basicData = fetchBasicData()
    
    let shouldFetchExtra = await checkCondition()
    
    if shouldFetchExtra {
        async let extraData = fetchExtraData()
        let (basic, extra) = await (basicData, extraData)
        process(basic: basic, extra: extra)
    } else {
        let basic = await basicData
        process(basic: basic)
    }
}
```

---

## Actors y @MainActor

### Actors - Thread Safety

```swift
actor BankAccount {
    private var balance: Double = 0
    
    func deposit(amount: Double) {
        balance += amount
    }
    
    func withdraw(amount: Double) -> Bool {
        guard balance >= amount else { return false }
        balance -= amount
        return true
    }
    
    func getBalance() -> Double {
        balance
    }
}

// Uso
let account = BankAccount()

Task {
    await account.deposit(amount: 100)
    let balance = await account.getBalance()
    print("Balance: \(balance)")
}
```

### @MainActor - UI Updates

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    
    func loadUsers() async {
        isLoading = true // Automáticamente en main thread
        
        do {
            users = try await fetchUsers()
        } catch {
            print("Error: \(error)")
        }
        
        isLoading = false
    }
}

// Funciones específicas en main thread
func updateUI() {
    Task { @MainActor in
        // Este código corre en main thread
        label.text = "Updated"
    }
}
```

### Nonisolated

```swift
actor DataManager {
    private var data: [String] = []
    
    // Puede ser llamado desde cualquier thread
    nonisolated func getDefaultData() -> [String] {
        return ["Default"]
    }
    
    // Requiere await
    func getData() -> [String] {
        return data
    }
}
```

---

## Async Sequences

Secuencias que producen valores de forma asíncrona:

### AsyncStream

```swift
func countdown(from: Int) -> AsyncStream<Int> {
    AsyncStream { continuation in
        Task {
            for i in (1...from).reversed() {
                continuation.yield(i)
                try await Task.sleep(nanoseconds: 1_000_000_000)
            }
            continuation.finish()
        }
    }
}

// Uso
Task {
    for await number in countdown(from: 5) {
        print(number) // 5, 4, 3, 2, 1
    }
}
```

### AsyncThrowingStream

```swift
func fetchUpdates() -> AsyncThrowingStream<Update, Error> {
    AsyncThrowingStream { continuation in
        let task = Task {
            while !Task.isCancelled {
                do {
                    let update = try await fetchNextUpdate()
                    continuation.yield(update)
                } catch {
                    continuation.finish(throwing: error)
                    return
                }
            }
            continuation.finish()
        }
        
        continuation.onTermination = { _ in
            task.cancel()
        }
    }
}

// Uso
Task {
    do {
        for try await update in fetchUpdates() {
            print("Update: \(update)")
        }
    } catch {
        print("Error: \(error)")
    }
}
```

### Map, Filter con AsyncSequence

```swift
extension AsyncSequence {
    func map<T>(_ transform: @escaping (Element) async -> T) -> AsyncMapSequence<Self, T> {
        AsyncMapSequence(self, transform: transform)
    }
}

// Uso
for await transformedValue in sequence.map { value in
    await transform(value)
} {
    print(transformedValue)
}
```

---

## Error Handling

### Try Await

```swift
func fetchAndDecode<T: Decodable>(_ type: T.Type, from url: URL) async throws -> T {
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw NetworkError.invalidResponse
    }
    
    return try JSONDecoder().decode(T.self, from: data)
}
```

### Multiple Try Await

```swift
func processData() async throws {
    let data1 = try await fetchData(from: url1)
    let data2 = try await fetchData(from: url2)
    
    // Si cualquiera falla, la función termina
    let combined = try await combineData(data1, data2)
}
```

### Try? with Async

```swift
func loadData() async {
    // Ignora errores, retorna nil si falla
    if let data = try? await fetchData() {
        process(data)
    }
}
```

---

## Migración de Callbacks

### Continuations - Puente entre Callbacks y Async

```swift
// Callback original
func fetchData(completion: @escaping (Result<Data, Error>) -> Void) {
    URLSession.shared.dataTask(with: url) { data, _, error in
        if let error = error {
            completion(.failure(error))
        } else if let data = data {
            completion(.success(data))
        }
    }.resume()
}

// Convertir a async/await
func fetchData() async throws -> Data {
    try await withCheckedThrowingContinuation { continuation in
        fetchData { result in
            continuation.resume(with: result)
        }
    }
}
```

### WithUnsafeContinuation

```swift
func legacyAPI(completion: @escaping (String) -> Void) {
    DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
        completion("Result")
    }
}

// Convertir a async
func modernAPI() async -> String {
    await withUnsafeContinuation { continuation in
        legacyAPI { result in
            continuation.resume(returning: result)
        }
    }
}
```

---

## Checklist de Aprendizaje

### Nivel Básico
- [ ] Entender qué es async/await y por qué usarlo
- [ ] Declarar funciones asíncronas con `async`
- [ ] Usar `await` para llamar funciones asíncronas
- [ ] Crear Tasks básicas
- [ ] Manejar errores con try/await

### Nivel Intermedio
- [ ] Usar async let para paralelismo
- [ ] Trabajar con TaskGroups
- [ ] Entender suspension points
- [ ] Implementar task cancellation
- [ ] Usar @MainActor para UI updates
- [ ] Crear AsyncSequences básicas

### Nivel Avanzado
- [ ] Implementar Actors propios
- [ ] Usar continuations para migrar callbacks
- [ ] Crear AsyncStreams complejas
- [ ] Optimizar performance con concurrencia estructurada
- [ ] Manejar race conditions
- [ ] Implementar custom async sequences

### Práctica
- [ ] Migrar API con callbacks a async/await
- [ ] Implementar sistema de caché con Actors
- [ ] Crear pipeline de datos con AsyncSequence
- [ ] Optimizar fetching paralelo de imágenes
- [ ] Implementar retry logic con async/await

---

## Próximos Pasos

### 1. Profundizar en Temas Relacionados
- **Actors** - Thread-safe state management
- **Sendable** - Thread-safe types
- **Combine** - Comparación con async/await
- **GCD** - Cuándo usar cada uno

### 2. Patrones Comunes
```swift
// Retry Pattern
func fetchWithRetry<T>(
    maxAttempts: Int = 3,
    operation: @escaping () async throws -> T
) async throws -> T {
    var lastError: Error?
    
    for attempt in 1...maxAttempts {
        do {
            return try await operation()
        } catch {
            lastError = error
            if attempt < maxAttempts {
                try await Task.sleep(nanoseconds: UInt64(attempt) * 1_000_000_000)
            }
        }
    }
    
    throw lastError ?? NetworkError.unknown
}

// Timeout Pattern
func withTimeout<T>(
    seconds: TimeInterval,
    operation: @escaping () async throws -> T
) async throws -> T {
    try await withThrowingTaskGroup(of: T.self) { group in
        group.addTask {
            try await operation()
        }
        
        group.addTask {
            try await Task.sleep(nanoseconds: UInt64(seconds * 1_000_000_000))
            throw TimeoutError()
        }
        
        let result = try await group.next()!
        group.cancelAll()
        return result
    }
}
```

### 3. Best Practices
- Preferir async/await sobre callbacks
- Usar Actors para estado mutable compartido
- Evitar blocking el thread con await
- Cancelar tasks apropiadamente
- Usar @MainActor para UI

### 4. Performance
- Minimizar suspension points innecesarios
- Usar async let para operaciones independientes
- Limitar concurrencia en TaskGroups
- Profile con Instruments

---

## Recursos

### Documentación Oficial
- 📚 [Concurrency - Swift Language Guide](https://docs.swift.org/swift-book/LanguageGuide/Concurrency.html)
- 📖 [Swift Concurrency](https://developer.apple.com/documentation/swift/swift_standard_library/concurrency)
- 🎓 [Meet async/await in Swift](https://developer.apple.com/videos/play/wwdc2021/10132/) - WWDC 2021

### Artículos
- 📝 [Swift Async/Await](https://www.hackingwithswift.com/swift/5.5/async-await) - Paul Hudson
- 📝 [Async/Await in Swift](https://www.swiftbysundell.com/articles/async-await/) - John Sundell
- 📝 [Actors in Swift](https://www.avanderlee.com/swift/actors/) - Antoine van der Lee
- 📝 [TaskGroup](https://www.donnywals.com/understanding-task-groups-in-swift/) - Donny Wals

### Videos
- 🎥 [Swift Concurrency Explained](https://www.youtube.com/watch?v=0P0KP08Pv9c) - Sean Allen
- 🎥 [Async/Await Tutorial](https://www.youtube.com/watch?v=lMKF7EWG2Cg) - Paul Hudson
- 🎥 [Actors & Sendable](https://www.youtube.com/watch?v=m-AzO7yQ0qA) - WWDC
- 🎥 [AsyncSequence](https://www.youtube.com/watch?v=n22F7KBNgz8) - Sean Allen

### Libros
- 📕 "Modern Concurrency in Swift" - Ray Wenderlich
- 📗 "Advanced Swift" - Concurrency chapter - objc.io
- 📘 "Async Await in Swift" - Paul Hudson

### Cursos
- 🎓 [Modern Swift Concurrency](https://www.udemy.com/course/swift-concurrency/) - Udemy
- 🎓 [Async/Await Masterclass](https://www.hackingwithswift.com/plus/async-await) - Hacking with Swift+
- 🎓 [Advanced Swift](https://www.linkedin.com/learning/advanced-swift) - LinkedIn Learning

### Herramientas
- 🛠️ [Instruments](https://developer.apple.com/instruments/) - Profiling
- 🛠️ [Thread Sanitizer](https://developer.apple.com/documentation/xcode/diagnosing-memory-thread-and-crash-issues-early) - Race condition detection
- 🛠️ [Swift Concurrency](https://github.com/apple/swift-evolution/blob/main/proposals/0296-async-await.md) - Evolution proposal

### Comunidad
- 💬 [Swift Forums - Concurrency](https://forums.swift.org/c/development/concurrency)
- 💬 [r/swift - Async/Await](https://reddit.com/r/swift)
- 💬 [Stack Overflow](https://stackoverflow.com/questions/tagged/swift+async-await)

---

**Anterior:** [Protocols & Extensions](Protocols-Extensions.md)  
**Próximo:** [Actors](Actors.md)
