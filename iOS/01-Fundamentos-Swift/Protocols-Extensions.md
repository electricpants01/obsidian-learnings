# Protocols & Extensions - Programación Orientada a Protocolos

## Tabla de Contenido
1. [¿Qué son los Protocols?](#qué-son-los-protocols)
2. [Protocol Syntax](#protocol-syntax)
3. [Protocol Requirements](#protocol-requirements)
4. [Protocol Inheritance](#protocol-inheritance)
5. [Protocol Composition](#protocol-composition)
6. [Extensions](#extensions)
7. [Protocol Extensions](#protocol-extensions)
8. [Associated Types](#associated-types)
9. [Casos de Uso Comunes](#casos-de-uso-comunes)
10. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
11. [Próximos Pasos](#próximos-pasos)
12. [Recursos](#recursos)

---

## ¿Qué son los Protocols?

Los **protocols** definen un contrato de métodos, propiedades y otros requisitos que adoptan tipos conformes (structs, classes, enums).

### Protocol vs Class

```swift
// ❌ Con herencia de clase (limitado)
class Vehicle {
    func start() {
        print("Starting vehicle")
    }
}

class Car: Vehicle { } // Solo puede heredar de UNA clase

// ✅ Con protocols (flexible)
protocol Drivable {
    func drive()
}

protocol Flyable {
    func fly()
}

struct FlyingCar: Drivable, Flyable { // Puede conformar MÚLTIPLES protocols
    func drive() { print("Driving") }
    func fly() { print("Flying") }
}
```

### Protocol-Oriented Programming (POP)

Swift favorece POP sobre OOP tradicional:

```swift
// Ejemplo: Sistema de autenticación
protocol Authenticatable {
    var isAuthenticated: Bool { get }
    func login(username: String, password: String)
    func logout()
}

// Diferentes implementaciones
struct EmailAuth: Authenticatable {
    var isAuthenticated: Bool = false
    
    func login(username: String, password: String) {
        print("Email login")
    }
    
    func logout() {
        print("Email logout")
    }
}

struct BiometricAuth: Authenticatable {
    var isAuthenticated: Bool = false
    
    func login(username: String, password: String) {
        print("Biometric login")
    }
    
    func logout() {
        print("Biometric logout")
    }
}
```

---

## Protocol Syntax

### Declaración Básica

```swift
protocol SomeProtocol {
    // Requisitos del protocol
}

// Conformidad
struct SomeStruct: SomeProtocol {
    // Implementación
}

// Class con protocol y superclass
class SomeClass: SuperClass, FirstProtocol, SecondProtocol {
    // Implementación
}
```

### Property Requirements

```swift
protocol Named {
    var name: String { get } // Solo lectura
    var fullName: String { get set } // Lectura y escritura
}

struct Person: Named {
    let name: String // get implícito con let
    var fullName: String // get y set con var
}

// También con computed properties
struct Company: Named {
    var name: String {
        return "Acme Corp"
    }
    
    var fullName: String {
        get { return "Acme Corporation" }
        set { /* ... */ }
    }
}
```

### Method Requirements

```swift
protocol Drawable {
    func draw()
    func draw(at point: CGPoint)
    mutating func move(to point: CGPoint) // mutating para structs
}

struct Circle: Drawable {
    var center: CGPoint
    
    func draw() {
        print("Drawing circle")
    }
    
    func draw(at point: CGPoint) {
        print("Drawing at \(point)")
    }
    
    mutating func move(to point: CGPoint) {
        center = point
    }
}
```

### Initializer Requirements

```swift
protocol Initializable {
    init(value: Int)
}

// Struct
struct MyStruct: Initializable {
    init(value: Int) {
        // Implementation
    }
}

// Class (requiere 'required')
class MyClass: Initializable {
    required init(value: Int) {
        // Implementation
    }
}
```

---

## Protocol Requirements

### Static Requirements

```swift
protocol Identifiable {
    static var idPrefix: String { get }
    static func createID() -> String
}

struct User: Identifiable {
    static var idPrefix: String = "USR"
    
    static func createID() -> String {
        return "\(idPrefix)_\(UUID().uuidString)"
    }
}

print(User.createID()) // "USR_xxx-xxx-xxx"
```

### Optional Requirements

```swift
@objc protocol DataSource {
    @objc optional func numberOfItems() -> Int
    @objc optional var itemTitle: String { get }
}

class MyDataSource: NSObject, DataSource {
    // Puede implementar o no los métodos opcionales
    func numberOfItems() -> Int {
        return 10
    }
    // itemTitle no implementado
}
```

### Protocol with Generic Constraints

```swift
protocol Container {
    associatedtype Item
    var count: Int { get }
    mutating func append(_ item: Item)
    subscript(i: Int) -> Item { get }
}

struct IntStack: Container {
    typealias Item = Int // Explícito (opcional)
    
    private var items: [Int] = []
    
    var count: Int {
        return items.count
    }
    
    mutating func append(_ item: Int) {
        items.append(item)
    }
    
    subscript(i: Int) -> Int {
        return items[i]
    }
}
```

---

## Protocol Inheritance

Los protocols pueden heredar de otros protocols:

```swift
protocol Named {
    var name: String { get }
}

protocol Aged {
    var age: Int { get }
}

// Protocol que hereda de múltiples protocols
protocol Person: Named, Aged {
    var address: String { get }
}

struct Student: Person {
    var name: String
    var age: Int
    var address: String
}
```

### Jerarquía de Protocols

```swift
protocol Vehicle {
    var wheels: Int { get }
    func start()
}

protocol MotorVehicle: Vehicle {
    var enginePower: Int { get }
    func accelerate()
}

protocol ElectricVehicle: MotorVehicle {
    var batteryCapacity: Int { get }
    func charge()
}

struct Tesla: ElectricVehicle {
    var wheels: Int = 4
    var enginePower: Int = 450
    var batteryCapacity: Int = 100
    
    func start() { print("Starting Tesla") }
    func accelerate() { print("Accelerating") }
    func charge() { print("Charging battery") }
}
```

---

## Protocol Composition

Combinar múltiples protocols sin crear un nuevo protocol:

```swift
protocol Named {
    var name: String { get }
}

protocol Aged {
    var age: Int { get }
}

// Función que requiere AMBOS protocols
func greet(_ person: Named & Aged) {
    print("Hello \(person.name), age \(person.age)")
}

struct Person: Named, Aged {
    var name: String
    var age: Int
}

let person = Person(name: "Ana", age: 25)
greet(person) // Funciona porque Person conforma ambos
```

### Con Classes

```swift
class Location {
    var latitude: Double
    var longitude: Double
    
    init(lat: Double, lon: Double) {
        latitude = lat
        longitude = lon
    }
}

protocol Named {
    var name: String { get }
}

// Requiere ser subclass de Location Y conformar Named
func show(_ place: Location & Named) {
    print("\(place.name) at \(place.latitude), \(place.longitude)")
}
```

---

## Extensions

Las extensions añaden funcionalidad a tipos existentes sin acceso al código fuente:

### Extension Básica

```swift
extension String {
    func capitalizedFirstLetter() -> String {
        return prefix(1).capitalized + dropFirst()
    }
    
    var isEmail: Bool {
        contains("@") && contains(".")
    }
}

let text = "hello world"
print(text.capitalizedFirstLetter()) // "Hello world"
print("user@example.com".isEmail)    // true
```

### Extension con Computed Properties

```swift
extension Int {
    var squared: Int {
        return self * self
    }
    
    var cubed: Int {
        return self * self * self
    }
}

print(5.squared) // 25
print(3.cubed)   // 27
```

### Extension con Methods

```swift
extension Array {
    func chunked(into size: Int) -> [[Element]] {
        stride(from: 0, to: count, by: size).map {
            Array(self[$0..<Swift.min($0 + size, count)])
        }
    }
}

let numbers = [1, 2, 3, 4, 5, 6, 7, 8]
print(numbers.chunked(into: 3)) // [[1, 2, 3], [4, 5, 6], [7, 8]]
```

### Extension with Initializers

```swift
extension CGPoint {
    init(x: Int, y: Int) {
        self.init(x: CGFloat(x), y: CGFloat(y))
    }
}

let point = CGPoint(x: 10, y: 20) // Ahora acepta Int
```

---

## Protocol Extensions

Proporcionar implementaciones por defecto para protocols:

### Default Implementation

```swift
protocol Greetable {
    var name: String { get }
    func greet()
}

// Default implementation
extension Greetable {
    func greet() {
        print("Hello, \(name)!")
    }
}

struct Person: Greetable {
    var name: String
    // No necesita implementar greet(), usa el default
}

let person = Person(name: "Ana")
person.greet() // "Hello, Ana!"
```

### Override Default Implementation

```swift
struct FormalPerson: Greetable {
    var name: String
    
    // Override del default
    func greet() {
        print("Good day, \(name).")
    }
}

let formal = FormalPerson(name: "Mr. Smith")
formal.greet() // "Good day, Mr. Smith."
```

### Conditional Conformance

```swift
extension Array: Greetable where Element: Greetable {
    var name: String {
        return "Group of \(count)"
    }
    
    func greet() {
        forEach { $0.greet() }
    }
}

let people = [
    Person(name: "Ana"),
    Person(name: "Luis")
]
people.greet()
// "Hello, Ana!"
// "Hello, Luis!"
```

### Protocol Extension con Constraints

```swift
extension Collection where Element: Numeric {
    func sum() -> Element {
        reduce(0, +)
    }
}

let numbers = [1, 2, 3, 4, 5]
print(numbers.sum()) // 15

let doubles = [1.5, 2.5, 3.0]
print(doubles.sum()) // 7.0
```

---

## Associated Types

Associated types permiten crear protocols genéricos:

### Básico

```swift
protocol Container {
    associatedtype Item
    
    var count: Int { get }
    mutating func append(_ item: Item)
    subscript(i: Int) -> Item { get }
}

struct Stack<Element>: Container {
    typealias Item = Element // Opcional, se infiere
    
    private var items: [Element] = []
    
    var count: Int {
        return items.count
    }
    
    mutating func append(_ item: Element) {
        items.append(item)
    }
    
    subscript(i: Int) -> Element {
        return items[i]
    }
}
```

### With Constraints

```swift
protocol SuffixableContainer: Container {
    associatedtype Suffix: SuffixableContainer where Suffix.Item == Item
    func suffix(_ size: Int) -> Suffix
}

extension Stack: SuffixableContainer {
    func suffix(_ size: Int) -> Stack {
        var result = Stack()
        for index in (count - size)..<count {
            result.append(self[index])
        }
        return result
    }
}
```

### Generic Where Clauses

```swift
extension Container where Item: Equatable {
    func contains(_ item: Item) -> Bool {
        for i in 0..<count {
            if self[i] == item {
                return true
            }
        }
        return false
    }
}

var stack = Stack<Int>()
stack.append(1)
stack.append(2)
print(stack.contains(2)) // true
```

---

## Casos de Uso Comunes

### 1. Dependency Injection

```swift
protocol NetworkServiceProtocol {
    func fetchData(from url: URL) async throws -> Data
}

class RealNetworkService: NetworkServiceProtocol {
    func fetchData(from url: URL) async throws -> Data {
        let (data, _) = try await URLSession.shared.data(from: url)
        return data
    }
}

class MockNetworkService: NetworkServiceProtocol {
    func fetchData(from url: URL) async throws -> Data {
        return Data() // Mock data
    }
}

class ViewModel {
    let networkService: NetworkServiceProtocol
    
    init(networkService: NetworkServiceProtocol) {
        self.networkService = networkService
    }
    
    func loadData() async {
        do {
            let data = try await networkService.fetchData(from: URL(string: "https://api.example.com")!)
            // Process data
        } catch {
            print("Error: \(error)")
        }
    }
}

// Production
let viewModel = ViewModel(networkService: RealNetworkService())

// Testing
let testViewModel = ViewModel(networkService: MockNetworkService())
```

### 2. Delegation Pattern

```swift
protocol TableViewCellDelegate: AnyObject {
    func cellDidTapButton(_ cell: TableViewCell)
}

class TableViewCell {
    weak var delegate: TableViewCellDelegate?
    
    func buttonTapped() {
        delegate?.cellDidTapButton(self)
    }
}

class ViewController: TableViewCellDelegate {
    func cellDidTapButton(_ cell: TableViewCell) {
        print("Button tapped in cell")
    }
}
```

### 3. Codable Extension

```swift
protocol APIResource: Codable {
    static var endpoint: String { get }
}

extension APIResource {
    static func fetch() async throws -> Self {
        let url = URL(string: "https://api.example.com/\(endpoint)")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode(Self.self, from: data)
    }
}

struct User: APIResource {
    static var endpoint: String = "users"
    
    let id: Int
    let name: String
}

// Uso
Task {
    let user = try await User.fetch()
}
```

### 4. Builder Pattern

```swift
protocol URLRequestBuilder {
    var baseURL: String { get }
    var path: String { get }
    var method: String { get }
    var headers: [String: String] { get }
    
    func build() -> URLRequest
}

extension URLRequestBuilder {
    func build() -> URLRequest {
        let url = URL(string: baseURL + path)!
        var request = URLRequest(url: url)
        request.httpMethod = method
        headers.forEach { request.setValue($1, forHTTPHeaderField: $0) }
        return request
    }
}

struct GetUsersRequest: URLRequestBuilder {
    var baseURL: String = "https://api.example.com"
    var path: String = "/users"
    var method: String = "GET"
    var headers: [String: String] = ["Content-Type": "application/json"]
}

let request = GetUsersRequest().build()
```

### 5. Equatable & Hashable

```swift
struct User: Equatable, Hashable {
    let id: Int
    let name: String
    let email: String
    
    // Swift auto-genera la implementación
}

let user1 = User(id: 1, name: "Ana", email: "ana@example.com")
let user2 = User(id: 1, name: "Ana", email: "ana@example.com")

print(user1 == user2) // true

// Usar en Set
var users: Set<User> = [user1, user2]
print(users.count) // 1 (solo uno porque son iguales)
```

---

## Checklist de Aprendizaje

### Nivel Básico
- [ ] Entender qué es un protocol y por qué usarlos
- [ ] Declarar protocols simples con properties y methods
- [ ] Hacer que structs/classes conformen protocols
- [ ] Usar protocols de la Standard Library (Equatable, Hashable)
- [ ] Crear extensions básicas para tipos existentes

### Nivel Intermedio
- [ ] Implementar protocol inheritance
- [ ] Usar protocol composition (&)
- [ ] Crear protocol extensions con default implementations
- [ ] Entender la diferencia entre protocol y class inheritance
- [ ] Aplicar protocols para dependency injection
- [ ] Usar protocols con generics

### Nivel Avanzado
- [ ] Implementar associated types
- [ ] Usar generic where clauses
- [ ] Crear conditional conformance
- [ ] Combinar protocols con extensions avanzadas
- [ ] Implementar Protocol-Oriented Programming patterns
- [ ] Optimizar código con protocol witnesses

### Práctica
- [ ] Refactorizar código OOP a POP
- [ ] Implementar un sistema de networking con protocols
- [ ] Crear un framework usando protocol-oriented design
- [ ] Implementar delegation pattern
- [ ] Crear custom Codable implementations

---

## Próximos Pasos

### 1. Profundizar en Temas Relacionados
- **Generics** - Combinar con protocols
- **Type Erasure** - AnySequence, AnyPublisher
- **Opaque Types** - some Protocol
- **Existential Types** - any Protocol

### 2. Patrones de Diseño
```swift
// Strategy Pattern con Protocols
protocol SortStrategy {
    func sort<T: Comparable>(_ array: [T]) -> [T]
}

struct BubbleSort: SortStrategy {
    func sort<T: Comparable>(_ array: [T]) -> [T] {
        var arr = array
        for i in 0..<arr.count {
            for j in 0..<arr.count - i - 1 {
                if arr[j] > arr[j + 1] {
                    arr.swapAt(j, j + 1)
                }
            }
        }
        return arr
    }
}

struct QuickSort: SortStrategy {
    func sort<T: Comparable>(_ array: [T]) -> [T] {
        guard array.count > 1 else { return array }
        let pivot = array[array.count / 2]
        let less = array.filter { $0 < pivot }
        let equal = array.filter { $0 == pivot }
        let greater = array.filter { $0 > pivot }
        return sort(less) + equal + sort(greater)
    }
}

class Sorter {
    var strategy: SortStrategy
    
    init(strategy: SortStrategy) {
        self.strategy = strategy
    }
    
    func sort<T: Comparable>(_ array: [T]) -> [T] {
        strategy.sort(array)
    }
}
```

### 3. Protocol Composition Avanzada
- Crear DSLs con protocols
- Type-safe builders
- Phantom types

### 4. Performance
- Protocol witness tables
- Static vs Dynamic dispatch
- Whole module optimization

### 5. Proyectos Sugeridos
- Crear un framework de networking type-safe
- Implementar un ORM simple
- Crear un sistema de validación extensible

---

## Recursos

### Documentación Oficial
- 📚 [Protocols - Swift Language Guide](https://docs.swift.org/swift-book/LanguageGuide/Protocols.html)
- 📖 [Extensions](https://docs.swift.org/swift-book/LanguageGuide/Extensions.html)
- 🎓 [Protocol-Oriented Programming](https://developer.apple.com/videos/play/wwdc2015/408/)

### Artículos y Tutoriales
- 📝 [Protocols in Swift](https://www.hackingwithswift.com/sixty/9/1/protocols) - Paul Hudson
- 📝 [Protocol Extensions](https://www.swiftbysundell.com/articles/protocol-extensions/) - John Sundell
- 📝 [Protocol-Oriented Programming](https://www.raywenderlich.com/6742901-protocol-oriented-programming-tutorial-in-swift) - Ray Wenderlich
- 📝 [Associated Types](https://www.swiftbysundell.com/articles/different-flavors-of-type-erasure-in-swift/) - John Sundell

### Videos
- 🎥 [Protocol-Oriented Programming in Swift](https://www.youtube.com/watch?v=g2LwFZatfTI) - WWDC 2015
- 🎥 [Swift Protocols Explained](https://www.youtube.com/watch?v=qbW2mJ5d2v4) - Sean Allen
- 🎥 [Protocol Extensions](https://www.youtube.com/watch?v=hJoJTD0tg4E) - Paul Hudson
- 🎥 [Mastering Protocols](https://www.youtube.com/watch?v=rdVbKLfGKwI) - iOS Academy

### Libros
- 📕 "Protocol-Oriented Programming in Swift" - Jon Hoffman
- 📗 "Swift Design Patterns" - Giordano Scalzo
- 📘 "Advanced Swift" - Protocol chapter - objc.io
- 📙 "Swift Programming" - Protocols - Big Nerd Ranch

### Cursos
- 🎓 [Protocol-Oriented Programming](https://www.linkedin.com/learning/protocol-oriented-programming-in-swift) - LinkedIn Learning
- 🎓 [Advanced iOS Development](https://www.udemy.com/course/advanced-ios/) - Udemy
- 🎓 [100 Days of Swift](https://www.hackingwithswift.com/100) - Paul Hudson

### Práctica
- 💻 [Swift Playgrounds](https://www.apple.com/swift/playgrounds/)
- 💻 [Exercism Swift - Protocols](https://exercism.org/tracks/swift)
- 💻 [HackerRank Swift](https://www.hackerrank.com/domains/swift)

### Herramientas
- 🛠️ [SwiftLint](https://github.com/realm/SwiftLint) - Protocol naming conventions
- 🛠️ [Sourcery](https://github.com/krzysztofzablocki/Sourcery) - Code generation
- 🛠️ [SwiftFormat](https://github.com/nicklockwood/SwiftFormat) - Code formatting

### Comunidad
- 💬 [Swift Forums - Protocols](https://forums.swift.org)
- 💬 [r/swift](https://reddit.com/r/swift)
- 💬 [Stack Overflow - Swift Protocols](https://stackoverflow.com/questions/tagged/swift+protocol)

---

**Anterior:** [Closures](Closures.md)  
**Próximo:** [Generics](Generics.md)
