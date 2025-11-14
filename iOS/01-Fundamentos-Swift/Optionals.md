# Optionals - Manejo Seguro de Valores Nulos

## Tabla de Contenido
1. [¿Qué son los Optionals?](#qué-son-los-optionals)
2. [Optional Binding](#optional-binding)
3. [Optional Chaining](#optional-chaining)
4. [Nil Coalescing Operator](#nil-coalescing-operator)
5. [Force Unwrapping](#force-unwrapping)
6. [Implicitly Unwrapped Optionals](#implicitly-unwrapped-optionals)
7. [Optional Map y FlatMap](#optional-map-y-flatmap)
8. [Casos de Uso Comunes](#casos-de-uso-comunes)
9. [Recursos](#recursos)

---

## ¿Qué son los Optionals?

Los Optionals son una característica fundamental de Swift que permite representar la **ausencia de un valor** de forma segura. Un optional puede contener un valor o ser `nil`.

### Sintaxis

```swift
// Optional String - puede ser String o nil
var nombre: String? = "Juan"
var apellido: String? = nil

// Optional Int
var edad: Int? = 25
var altura: Double? = nil

// Optional con inferencia de tipo
let resultado: Int? = Int("123") // Optional(123)
let fallo: Int? = Int("abc")     // nil
```

### ¿Por qué son importantes?

```swift
// ❌ Sin Optionals (otros lenguajes)
String nombre = null;  // NullPointerException en runtime!

// ✅ Con Optionals (Swift)
var nombre: String? = nil
// El compilador te obliga a manejar el caso nil
```

### Anatomía de un Optional

```swift
enum Optional<Wrapped> {
    case none           // nil
    case some(Wrapped)  // contiene un valor
}

// Cuando declaras:
var age: Int? = 25

// Swift lo trata como:
var age: Optional<Int> = .some(25)

// nil es:
var name: String? = nil
// Es equivalente a:
var name: Optional<String> = .none
```

---

## Optional Binding

### if let

La forma más común y segura de unwrap optionals:

```swift
var username: String? = "carlos123"

// ✅ Safe unwrapping
if let name = username {
    print("Usuario: \(name)")
    // 'name' es String (no opcional)
} else {
    print("No hay usuario")
}

// Multiple optional binding
var email: String? = "user@example.com"
var password: String? = "secret123"

if let email = email, let password = password {
    print("Email: \(email)")
    print("Password: \(password)")
    // Ambos disponibles aquí
}

// Con condiciones adicionales
if let age = age, age >= 18 {
    print("Mayor de edad: \(age)")
}
```

### guard let

Para validación temprana y salida anticipada:

```swift
func processUser(name: String?, age: Int?) {
    guard let name = name else {
        print("Nombre requerido")
        return
    }
    
    guard let age = age else {
        print("Edad requerida")
        return
    }
    
    // name y age disponibles en todo el scope
    print("\(name) tiene \(age) años")
}

// Ejemplo realista
func login(email: String?, password: String?) -> Bool {
    guard let email = email, !email.isEmpty else {
        print("Email inválido")
        return false
    }
    
    guard let password = password, password.count >= 6 else {
        print("Password debe tener al menos 6 caracteres")
        return false
    }
    
    // Proceder con login
    print("Logging in with: \(email)")
    return true
}
```

### while let

Para iterar mientras haya valores:

```swift
var numbers: [Int?] = [1, 2, nil, 3, 4, nil, 5]
var index = 0

while index < numbers.count, let number = numbers[index] {
    print("Número: \(number)")
    index += 1
}

// Uso con Iterator
var iterator = [1, 2, 3].makeIterator()
while let value = iterator.next() {
    print(value)
}
```

---

## Optional Chaining

Permite acceder a propiedades, métodos y subscripts de optionals de forma segura:

```swift
class Person {
    var residence: Residence?
}

class Residence {
    var address: Address?
    var numberOfRooms = 1
}

class Address {
    var street: String?
    var city: String?
    var zipCode: String?
}

let person = Person()

// ✅ Optional Chaining - retorna nil si cualquier parte es nil
let street = person.residence?.address?.street
// street es String? (nil en este caso)

let rooms = person.residence?.numberOfRooms
// rooms es Int? (nil porque residence es nil)

// Sin optional chaining necesitarías:
var streetManual: String?
if let residence = person.residence {
    if let address = residence.address {
        streetManual = address.street
    }
}
```

### Optional Chaining con Métodos

```swift
class User {
    var profile: Profile?
}

class Profile {
    func updateInfo() {
        print("Información actualizada")
    }
    
    func getName() -> String {
        return "John Doe"
    }
}

let user = User()

// Llamar método - no hace nada si profile es nil
user.profile?.updateInfo() // Void?

// Obtener resultado del método
let name = user.profile?.getName() // String?
```

### Optional Chaining con Subscripts

```swift
var addresses = ["home": "123 Main St", "work": "456 Office Blvd"]
var primaryAddress: [String: String]? = addresses

// Acceso seguro a subscript
let homeAddress = primaryAddress?["home"] // String??

// Array opcional
var numbers: [Int]? = [1, 2, 3]
let first = numbers?[0] // Int?
let outOfBounds = numbers?[10] // Int? (nil)
```

---

## Nil Coalescing Operator

El operador `??` proporciona un valor por defecto cuando un optional es nil:

```swift
let defaultName = "Guest"
var username: String? = nil

// ✅ Nil coalescing
let displayName = username ?? defaultName
print(displayName) // "Guest"

username = "carlos"
let newDisplayName = username ?? defaultName
print(newDisplayName) // "carlos"

// Encadenamiento
let name: String? = nil
let surname: String? = nil
let fullName = name ?? surname ?? "Anónimo"

// Con optional chaining
class Settings {
    var theme: String?
}

var settings: Settings? = nil
let theme = settings?.theme ?? "Default"
```

### Nil Coalescing con Expresiones

```swift
// Con operaciones
let price: Double? = nil
let finalPrice = (price ?? 0) * 1.16

// Con funciones
func getDefaultUser() -> String {
    print("Creando usuario por defecto")
    return "guest_user"
}

let user = username ?? getDefaultUser()
// Solo llama getDefaultUser() si username es nil
```

---

## Force Unwrapping

### ⚠️ Cuándo es peligroso

```swift
var name: String? = nil

// ❌ CRASH! Fatal error: Unexpectedly found nil
// let forcedName = name!
```

### ✅ Cuándo es aceptable

```swift
// 1. Después de verificar explícitamente
if name != nil {
    print(name!) // Seguro aquí
}

// 2. Con guard
func process(value: Int?) {
    guard value != nil else { return }
    print(value!) // Seguro después de guard
}

// 3. En fatalError paths
guard let config = loadConfig() else {
    fatalError("Configuración crítica no encontrada")
}

// 4. En tests
func testUserCreation() {
    let user = createUser()
    XCTAssertNotNil(user)
    XCTAssertEqual(user!.name, "Test User") // OK en tests
}
```

### Regla de Oro

```swift
// ❌ NUNCA hagas esto
let value = dictionary[key]!

// ✅ Siempre haz esto
if let value = dictionary[key] {
    // Usar value
}

// o
let value = dictionary[key] ?? defaultValue
```

---

## Implicitly Unwrapped Optionals

### Declaración

```swift
var username: String! = "carlos"
// Puede ser nil pero se desenvuelve automáticamente

print(username) // "carlos" - No necesita unwrapping
```

### Cuándo usar IUO

```swift
// 1. IBOutlets en UIKit
@IBOutlet weak var titleLabel: UILabel!

// 2. Propiedades que se inicializan después del init
class ViewController: UIViewController {
    var viewModel: ViewModel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        viewModel = ViewModel() // Siempre se inicializa aquí
    }
}

// 3. Dependency Injection
class Service {
    var apiClient: APIClient!
    
    func inject(client: APIClient) {
        self.apiClient = client
    }
}
```

### ⚠️ Peligros

```swift
var value: String! = nil
// Esto compila pero crashea en runtime:
// print(value)

// Mejor usar optional normal
var value: String? = nil
if let value = value {
    print(value)
}
```

---

## Optional Map y FlatMap

### map - Transformar el valor si existe

```swift
let numberString: String? = "42"

// Sin map
var doubled: Int?
if let number = Int(numberString) {
    doubled = number * 2
}

// Con map
let doubledMap = numberString.flatMap { Int($0) }.map { $0 * 2 }

// Ejemplo más claro
struct User {
    let name: String
    let age: Int
}

let user: User? = User(name: "Ana", age: 25)

// Obtener edad + 1
let nextAge = user.map { $0.age + 1 } // Optional(26)

// Cadena de transformaciones
let ageString = user
    .map { $0.age }
    .map { $0 + 1 }
    .map { String($0) } // Optional("26")
```

### flatMap - Aplanar optionals anidados

```swift
let numberString: String? = "42"

// map retorna Optional<Optional<Int>>
let nestedOptional = numberString.map { Int($0) } // Int??

// flatMap retorna Optional<Int>
let flatOptional = numberString.flatMap { Int($0) } // Int?

// Ejemplo práctico
func getUserId() -> String? {
    return "user_123"
}

func fetchUser(id: String) -> User? {
    return User(name: "Carlos", age: 30)
}

// Con flatMap
let user = getUserId().flatMap { fetchUser(id: $0) }

// Sin flatMap (más verboso)
var userManual: User?
if let id = getUserId() {
    userManual = fetchUser(id: id)
}
```

---

## Casos de Uso Comunes

### 1. Conversión de Strings

```swift
let input = "42"
let number = Int(input) // Int?

if let num = number {
    print("Número válido: \(num)")
} else {
    print("Conversión falló")
}

// Con guard
func processInput(_ input: String) {
    guard let number = Int(input) else {
        print("Input inválido")
        return
    }
    
    print("Procesando: \(number)")
}
```

### 2. Dictionary Access

```swift
let scores = ["Ana": 95, "Luis": 87, "María": 92]

// Acceso seguro
if let anaScore = scores["Ana"] {
    print("Ana: \(anaScore)")
}

// Con default
let unknownScore = scores["Pedro"] ?? 0

// Iterar solo valores válidos
let names = ["Ana", "Luis", "Pedro", "María"]
let validScores = names.compactMap { scores[$0] }
print(validScores) // [95, 87, 92]
```

### 3. Networking Responses

```swift
struct APIResponse {
    let data: Data?
    let error: Error?
}

func handleResponse(_ response: APIResponse) {
    if let error = response.error {
        print("Error: \(error)")
        return
    }
    
    guard let data = response.data else {
        print("No data")
        return
    }
    
    // Process data
}
```

### 4. Optional Protocol Conformance

```swift
protocol Drivable {
    func drive()
}

class Car: Drivable {
    func drive() {
        print("Driving car")
    }
}

let vehicle: Any? = Car()

// Conditional casting
if let drivable = vehicle as? Drivable {
    drivable.drive()
}
```

### 5. First/Last en Collections

```swift
let numbers = [1, 2, 3, 4, 5]

if let first = numbers.first {
    print("First: \(first)")
}

if let last = numbers.last {
    print("Last: \(last)")
}

// Empty array
let empty: [Int] = []
print(empty.first) // nil
print(empty.last)  // nil
```

---

## Mejores Prácticas

### ✅ DO

```swift
// Usa optional binding
if let value = optional {
    use(value)
}

// Usa guard para validación temprana
guard let value = optional else { return }

// Usa nil coalescing para defaults
let value = optional ?? defaultValue

// Usa optional chaining
let property = object?.property?.subProperty

// Usa compactMap para filtrar nils
let validItems = items.compactMap { $0 }
```

### ❌ DON'T

```swift
// No uses force unwrap sin verificar
// let value = optional! // PELIGROSO

// No uses IUO sin razón
// var value: String! = nil

// No anides demasiados ifs
// if let a = a {
//     if let b = b {
//         if let c = c {
//             // Demasiado anidado
//         }
//     }
// }

// Mejor: usa múltiple binding
if let a = a, let b = b, let c = c {
    // Más limpio
}
```

---

## Recursos

### Documentación Oficial
- 📚 [Optionals - Swift Language Guide](https://docs.swift.org/swift-book/LanguageGuide/TheBasics.html#ID330)
- 📖 [Optional Protocol](https://developer.apple.com/documentation/swift/optional)
- 🎓 [Error Handling](https://docs.swift.org/swift-book/LanguageGuide/ErrorHandling.html)

### Artículos y Tutoriales
- 📝 [Optionals in Swift Explained](https://www.hackingwithswift.com/sixty/10/1/handling-missing-data) - Paul Hudson
- 📝 [Optional Chaining Guide](https://www.swiftbysundell.com/articles/optional-chaining/) - John Sundell
- 📝 [Guard vs If Let](https://www.avanderlee.com/swift/guard-let-vs-if-let/) - Antoine van der Lee

### Videos
- 🎥 [Swift Optionals Explained](https://www.youtube.com/watch?v=7a7As0uNWOQ) - Sean Allen
- 🎥 [Mastering Optionals](https://www.youtube.com/watch?v=YH3t4YF7gq8) - Paul Hudson
- 🎥 [Optional Chaining Tutorial](https://www.youtube.com/watch?v=2ksXKLPEDPI) - CodeWithChris

### Libros
- 📕 "Swift Programming" - Capítulo sobre Optionals - Big Nerd Ranch
- 📗 "iOS Programming Fundamentals with Swift" - Matt Neuburg
- 📘 "Advanced Swift" - objc.io

### Práctica
- 💻 [Swift Optionals Exercises](https://exercism.org/tracks/swift) - Exercism
- 💻 [HackerRank Swift](https://www.hackerrank.com/domains/tutorials/30-days-of-code)
- 💻 [LeetCode Swift Problems](https://leetcode.com)

### Herramientas
- 🛠️ [SwiftLint](https://github.com/realm/SwiftLint) - Detecta force unwraps
- 🛠️ [Xcode Warnings](https://developer.apple.com/documentation/xcode) - Avisos del compilador

---

**Anterior:** [Swift Basics](Swift-Basics.md)  
**Próximo:** [Closures](Closures.md)
