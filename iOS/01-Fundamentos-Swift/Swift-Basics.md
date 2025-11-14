# Swift Basics - Fundamentos de Swift

## Tabla de Contenido
1. [Variables y Constantes](#variables-y-constantes)
2. [Tipos de Datos](#tipos-de-datos)
3. [Colecciones](#colecciones)
4. [Control de Flujo](#control-de-flujo)
5. [Funciones](#funciones)
6. [Closures](#closures-introducción)
7. [Enumeraciones](#enumeraciones)
8. [Structs vs Classes](#structs-vs-classes)
9. [Recursos](#recursos)

---

## Variables y Constantes

### Variables (var) - Mutables
```swift
var nombre = "Juan"
nombre = "Pedro" // ✅ Permitido

var edad = 25
edad = 26 // ✅ Permitido

var precio: Double = 19.99
precio = 29.99 // ✅ Permitido
```

### Constantes (let) - Inmutables (Preferidas)
```swift
let pi = 3.14159
// pi = 3.14 // ❌ Error: Cannot assign to value

let maxIntentos = 3
// maxIntentos = 5 // ❌ Error

let appName = "Mi App iOS"
```

### ¿Cuándo usar cada una?

**Usa `let` por defecto:**
- Valores que no cambiarán
- Mejora el rendimiento
- Hace el código más seguro y predecible
- Swift optimiza mejor las constantes

**Usa `var` solo cuando:**
- El valor necesita cambiar
- Estás acumulando datos
- Necesitas mutabilidad explícita

```swift
// ✅ Buena práctica
let userID = UUID()
let createdAt = Date()
var loginAttempts = 0 // Necesita incrementarse

// ❌ Mala práctica
var userID = UUID() // No cambiará, debería ser let
```

---

## Tipos de Datos

### Tipos Básicos

```swift
// Enteros
let age: Int = 25
let smallNumber: Int8 = 127
let largeNumber: Int64 = 9_223_372_036_854_775_807

// Números decimales
let pi: Double = 3.14159 // 64-bit (más precisión)
let price: Float = 9.99  // 32-bit (menos memoria)

// Booleanos
let isLoggedIn: Bool = true
let hasPermission: Bool = false

// Strings
let name: String = "Carlos"
let multiline = """
    Esta es una cadena
    de múltiples líneas
    en Swift
    """

// Caracteres
let letter: Character = "A"
let emoji: Character = "😊"
```

### Inferencia de Tipos

```swift
// Swift infiere el tipo automáticamente
let number = 42        // Int
let decimal = 3.14     // Double
let text = "Hello"     // String
let flag = true        // Bool

// Explicit typing cuando es necesario
let temperature: Double = 98 // Queremos Double, no Int
let coordinates: (Double, Double) = (37.7749, -122.4194)
```

### Type Aliases

```swift
typealias UserID = String
typealias Coordinate = (latitude: Double, longitude: Double)

let user: UserID = "user_12345"
let location: Coordinate = (37.7749, -122.4194)

print(location.latitude)  // 37.7749
print(location.longitude) // -122.4194
```

### Conversión de Tipos

```swift
let intNumber = 42
let doubleNumber = Double(intNumber) // 42.0

let price = 19.99
let roundedPrice = Int(price) // 19 (trunca, no redondea)

let numberString = "123"
if let number = Int(numberString) {
    print("Número convertido: \(number)")
}

// String interpolation
let age = 25
let message = "Tengo \(age) años"
let calculation = "2 + 2 = \(2 + 2)"
```

---

## Colecciones

### Arrays

```swift
// Declaración
var numbers: [Int] = [1, 2, 3, 4, 5]
var names = ["Ana", "Luis", "María"] // Tipo inferido: [String]

// Acceso
let firstNumber = numbers[0] // 1
let lastName = names[2]      // "María"

// Modificación
numbers.append(6)           // [1, 2, 3, 4, 5, 6]
numbers.insert(0, at: 0)    // [0, 1, 2, 3, 4, 5, 6]
numbers.remove(at: 0)       // [1, 2, 3, 4, 5, 6]

// Propiedades
print(numbers.count)        // 6
print(numbers.isEmpty)      // false
print(numbers.first)        // Optional(1)
print(numbers.last)         // Optional(6)

// Iteración
for number in numbers {
    print(number)
}

for (index, number) in numbers.enumerated() {
    print("Index \(index): \(number)")
}

// Métodos útiles
let doubled = numbers.map { $0 * 2 }
let evens = numbers.filter { $0 % 2 == 0 }
let sum = numbers.reduce(0, +)
let sorted = numbers.sorted()
let reversed = numbers.reversed()

// Array multidimensional
let matrix: [[Int]] = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]
print(matrix[1][1]) // 5
```

### Diccionarios

```swift
// Declaración
var ages: [String: Int] = ["Juan": 25, "María": 30]
var scores = ["Ana": 95, "Luis": 87] // Tipo inferido

// Acceso
let juanAge = ages["Juan"] // Optional(25)
if let age = ages["María"] {
    print("María tiene \(age) años")
}

// Modificación
ages["Pedro"] = 28          // Agregar
ages["Juan"] = 26           // Actualizar
ages["Pedro"] = nil         // Eliminar

// Propiedades
print(ages.count)           // 2
print(ages.isEmpty)         // false
print(ages.keys)            // ["Juan", "María"]
print(ages.values)          // [26, 30]

// Iteración
for (name, age) in ages {
    print("\(name): \(age) años")
}

// Default value
let unknownAge = ages["Desconocido", default: 0]

// Dictionary complejo
struct User {
    let id: String
    let name: String
}

var users: [String: User] = [
    "user1": User(id: "user1", name: "Ana"),
    "user2": User(id: "user2", name: "Luis")
]
```

### Sets

```swift
// Declaración
var uniqueNumbers: Set<Int> = [1, 2, 3, 3, 4, 4, 5]
print(uniqueNumbers) // [1, 2, 3, 4, 5] (orden no garantizado)

// Operaciones
uniqueNumbers.insert(6)
uniqueNumbers.remove(1)
let contains = uniqueNumbers.contains(3) // true

// Operaciones de conjuntos
let setA: Set = [1, 2, 3, 4]
let setB: Set = [3, 4, 5, 6]

let union = setA.union(setB)                // [1, 2, 3, 4, 5, 6]
let intersection = setA.intersection(setB)  // [3, 4]
let difference = setA.subtracting(setB)     // [1, 2]
let symmetric = setA.symmetricDifference(setB) // [1, 2, 5, 6]

// Verificación de subconjuntos
let isSubset = setA.isSubset(of: setB)
let isSuperset = setA.isSuperset(of: setB)
```

---

## Control de Flujo

### If-Else

```swift
let age = 18

if age >= 18 {
    print("Mayor de edad")
} else if age >= 13 {
    print("Adolescente")
} else {
    print("Menor de edad")
}

// Multiple conditions
let isWeekend = true
let isRaining = false

if isWeekend && !isRaining {
    print("Perfecto para salir")
}

// Ternary operator
let message = age >= 18 ? "Adulto" : "Menor"
```

### Guard

```swift
func processUser(name: String?) {
    guard let name = name else {
        print("Nombre inválido")
        return
    }
    
    // name está disponible aquí
    print("Procesando usuario: \(name)")
}

// Multiple guards
func validateUser(email: String?, password: String?) -> Bool {
    guard let email = email, !email.isEmpty else {
        return false
    }
    
    guard let password = password, password.count >= 6 else {
        return false
    }
    
    return true
}

// Guard con condiciones
func divide(_ a: Int, by b: Int) -> Int? {
    guard b != 0 else {
        print("No se puede dividir por cero")
        return nil
    }
    return a / b
}
```

### Switch

```swift
let dayOfWeek = "Lunes"

switch dayOfWeek {
case "Lunes", "Martes", "Miércoles", "Jueves", "Viernes":
    print("Día laboral")
case "Sábado", "Domingo":
    print("Fin de semana")
default:
    print("Día inválido")
}

// Switch con rangos
let score = 85

switch score {
case 90...100:
    print("Excelente")
case 80..<90:
    print("Muy bueno")
case 70..<80:
    print("Bueno")
case 60..<70:
    print("Regular")
default:
    print("Necesita mejorar")
}

// Switch con tuplas
let point = (x: 0, y: 0)

switch point {
case (0, 0):
    print("Origen")
case (let x, 0):
    print("En el eje X en \(x)")
case (0, let y):
    print("En el eje Y en \(y)")
case (let x, let y) where x == y:
    print("En la diagonal en (\(x), \(y))")
case (let x, let y):
    print("En (\(x), \(y))")
}

// Switch con enums
enum Direction {
    case north, south, east, west
}

let direction: Direction = .north

switch direction {
case .north:
    print("Ir al norte")
case .south:
    print("Ir al sur")
case .east:
    print("Ir al este")
case .west:
    print("Ir al oeste")
}
```

### Loops

```swift
// For-in
for i in 1...5 {
    print(i) // 1, 2, 3, 4, 5
}

for i in 1..<5 {
    print(i) // 1, 2, 3, 4
}

// Stride
for i in stride(from: 0, to: 10, by: 2) {
    print(i) // 0, 2, 4, 6, 8
}

// While
var count = 0
while count < 5 {
    print(count)
    count += 1
}

// Repeat-while (do-while)
var number = 0
repeat {
    print(number)
    number += 1
} while number < 5

// Break y Continue
for i in 1...10 {
    if i == 5 {
        continue // Salta este iteration
    }
    if i == 8 {
        break // Sale del loop
    }
    print(i)
}

// Labeled statements
outerLoop: for i in 1...3 {
    for j in 1...3 {
        if i * j > 4 {
            break outerLoop
        }
        print("\(i) x \(j) = \(i * j)")
    }
}
```

---

## Funciones

### Funciones Básicas

```swift
// Sin parámetros, sin retorno
func greet() {
    print("Hello!")
}

// Con parámetros
func greet(name: String) {
    print("Hello, \(name)!")
}

// Con retorno
func add(_ a: Int, _ b: Int) -> Int {
    return a + b
}

// Return implícito (single expression)
func multiply(_ a: Int, _ b: Int) -> Int {
    a * b
}

// Múltiples parámetros
func calculate(a: Int, b: Int, operation: String) -> Int {
    switch operation {
    case "+": return a + b
    case "-": return a - b
    case "*": return a * b
    case "/": return b != 0 ? a / b : 0
    default: return 0
    }
}
```

### Argument Labels

```swift
// External y internal names
func greet(person name: String, from hometown: String) {
    print("Hello \(name) from \(hometown)!")
}
greet(person: "Ana", from: "Madrid")

// Omitir label con _
func sum(_ a: Int, _ b: Int) -> Int {
    a + b
}
sum(5, 3) // Sin labels

// Valores por defecto
func greet(name: String = "Guest") {
    print("Hello, \(name)!")
}
greet()           // "Hello, Guest!"
greet(name: "Ana") // "Hello, Ana!"
```

### Parámetros Variádicos

```swift
func average(_ numbers: Double...) -> Double {
    guard !numbers.isEmpty else { return 0 }
    let sum = numbers.reduce(0, +)
    return sum / Double(numbers.count)
}

print(average(1, 2, 3, 4, 5)) // 3.0
print(average(10, 20))         // 15.0
```

### Inout Parameters

```swift
func swapValues(_ a: inout Int, _ b: inout Int) {
    let temp = a
    a = b
    b = temp
}

var x = 5
var y = 10
swapValues(&x, &y)
print("x: \(x), y: \(y)") // x: 10, y: 5
```

### Function Types

```swift
// Función como tipo
func add(_ a: Int, _ b: Int) -> Int {
    a + b
}

let mathOperation: (Int, Int) -> Int = add
print(mathOperation(5, 3)) // 8

// Función como parámetro
func performOperation(_ a: Int, _ b: Int, operation: (Int, Int) -> Int) -> Int {
    operation(a, b)
}

let result = performOperation(10, 5, operation: add)

// Función como retorno
func makeIncrementer(by amount: Int) -> (Int) -> Int {
    func increment(_ value: Int) -> Int {
        value + amount
    }
    return increment
}

let incrementByTwo = makeIncrementer(by: 2)
print(incrementByTwo(5)) // 7
```

---

## Closures (Introducción)

```swift
// Sintaxis completa
let greeting = { (name: String) -> String in
    return "Hello, \(name)!"
}

// Sintaxis simplificada
let add = { (a: Int, b: Int) -> Int in
    a + b
}

// Como parámetro
func performTwice(_ operation: () -> Void) {
    operation()
    operation()
}

performTwice {
    print("Ejecutando operación")
}

// Shorthand argument names
let numbers = [1, 2, 3, 4, 5]
let doubled = numbers.map { $0 * 2 }
let evens = numbers.filter { $0 % 2 == 0 }
```

---

## Enumeraciones

```swift
// Enum básico
enum Direction {
    case north
    case south
    case east
    case west
}

let heading: Direction = .north

// Multiple cases en una línea
enum Planet {
    case mercury, venus, earth, mars
}

// Raw values
enum StatusCode: Int {
    case success = 200
    case notFound = 404
    case serverError = 500
}

let code = StatusCode.success
print(code.rawValue) // 200

// Associated values
enum Barcode {
    case upc(Int, Int, Int, Int)
    case qrCode(String)
}

var productBarcode = Barcode.upc(8, 85909, 51226, 3)
productBarcode = .qrCode("ABCDEFGHIJKLMNOP")

// Pattern matching
switch productBarcode {
case .upc(let numberSystem, let manufacturer, let product, let check):
    print("UPC: \(numberSystem), \(manufacturer), \(product), \(check)")
case .qrCode(let productCode):
    print("QR code: \(productCode)")
}

// Enum con métodos
enum Suit {
    case spades, hearts, diamonds, clubs
    
    func description() -> String {
        switch self {
        case .spades: return "♠️"
        case .hearts: return "♥️"
        case .diamonds: return "♦️"
        case .clubs: return "♣️"
        }
    }
}
```

---

## Structs vs Classes

### Struct (Value Type)

```swift
struct Point {
    var x: Double
    var y: Double
    
    // Memberwise initializer automático
    // init(x: Double, y: Double)
    
    func distance(to other: Point) -> Double {
        let dx = x - other.x
        let dy = y - other.y
        return sqrt(dx * dx + dy * dy)
    }
    
    mutating func move(by dx: Double, dy: Double) {
        x += dx
        y += dy
    }
}

var point1 = Point(x: 0, y: 0)
var point2 = point1 // Copia
point2.x = 10

print(point1.x) // 0 (no afectado)
print(point2.x) // 10
```

### Class (Reference Type)

```swift
class Person {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    
    func greet() {
        print("Hello, my name is \(name)")
    }
}

let person1 = Person(name: "Ana", age: 25)
let person2 = person1 // Referencia
person2.name = "Luis"

print(person1.name) // "Luis" (afectado)
print(person2.name) // "Luis"
```

### ¿Cuándo usar cada uno?

**Usa Struct cuando:**
- Encapsulas datos simples
- Quieres copias independientes
- Los datos son relativamente pequeños
- No necesitas herencia

**Usa Class cuando:**
- Necesitas herencia
- Necesitas identidad de objeto
- Quieres compartir estado
- Trabajas con Objective-C

---

## Recursos

### Documentación Oficial
- 📚 [The Swift Programming Language](https://docs.swift.org/swift-book/) - Libro oficial de Swift
- 📖 [Swift.org Documentation](https://swift.org/documentation/) - Documentación completa
- 🎓 [Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/) - Guías de diseño

### Cursos Recomendados
- 🎥 [Stanford CS193p](https://cs193p.sites.stanford.edu/) - Curso de Stanford (Gratis)
- 🎥 [100 Days of SwiftUI](https://www.hackingwithswift.com/100/swiftui) - Paul Hudson
- 🎥 [Swift Fundamentals](https://www.udemy.com/course/ios-13-app-development-bootcamp/) - Angela Yu (Udemy)

### Libros
- 📕 "Swift Programming: The Big Nerd Ranch Guide" - Matthew Mathias
- 📗 "iOS Programming Fundamentals with Swift" - Matt Neuburg
- 📘 "Advanced Swift" - Chris Eidhof & Airspeed Velocity

### Tutoriales y Artículos
- 🌐 [Hacking with Swift](https://www.hackingwithswift.com) - Paul Hudson
- 🌐 [Swift by Sundell](https://www.swiftbysundell.com) - John Sundell
- 🌐 [NSHipster](https://nshipster.com) - Mattt Thompson

### Videos YouTube
- 📺 [Sean Allen](https://www.youtube.com/c/SeanAllen) - iOS Development
- 📺 [Paul Hudson](https://www.youtube.com/c/PaulHudson) - Swift tutorials
- 📺 [Kavsoft](https://www.youtube.com/c/Kavsoft) - SwiftUI

### Práctica
- 💻 [Swift Playgrounds](https://www.apple.com/swift/playgrounds/) - App para aprender Swift
- 💻 [HackerRank Swift](https://www.hackerrank.com/domains/tutorials/30-days-of-code) - Ejercicios
- 💻 [LeetCode](https://leetcode.com) - Problemas algorítmicos en Swift

### Comunidades
- 💬 [Swift Forums](https://forums.swift.org) - Comunidad oficial
- 💬 [r/swift](https://reddit.com/r/swift) - Reddit
- 💬 [iOS Developers Slack](https://ios-developers.io) - Slack community
- 💬 [Swift Twitter](https://twitter.com/search?q=%23swiftlang) - Twitter

---

**Próximo tema:** [Optionals](Optionals.md) - Manejo seguro de valores nulos
