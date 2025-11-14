# Closures - Funciones como Ciudadanos de Primera Clase

## Tabla de Contenido
1. [¿Qué son los Closures?](#qué-son-los-closures)
2. [Sintaxis de Closures](#sintaxis-de-closures)
3. [Trailing Closures](#trailing-closures)
4. [Capturing Values](#capturing-values)
5. [Escaping Closures](#escaping-closures)
6. [Autoclosures](#autoclosures)
7. [Map, Filter, Reduce](#map-filter-reduce)
8. [Casos de Uso Comunes](#casos-de-uso-comunes)
9. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
10. [Próximos Pasos](#próximos-pasos)
11. [Recursos](#recursos)

---

## ¿Qué son los Closures?

Los **closures** son bloques de código auto-contenidos que pueden ser pasados y usados en tu código. Son similares a lambdas en otros lenguajes o blocks en Objective-C.

### Tres Formas de Closures

```swift
// 1. Funciones globales (son closures con nombre)
func add(a: Int, b: Int) -> Int {
    a + b
}

// 2. Funciones anidadas (closures con nombre dentro de otra función)
func makeIncrementer(incrementAmount: Int) -> () -> Int {
    var total = 0
    func incrementer() -> Int {
        total += incrementAmount
        return total
    }
    return incrementer
}

// 3. Expresiones de closure (closures sin nombre/anónimos)
let multiply = { (a: Int, b: Int) -> Int in
    return a * b
}
```

### ¿Por qué son importantes?

```swift
// Permiten código más expresivo y funcional
let numbers = [1, 2, 3, 4, 5]

// Sin closures (imperativo)
var doubled: [Int] = []
for number in numbers {
    doubled.append(number * 2)
}

// Con closures (funcional)
let doubledClosure = numbers.map { $0 * 2 }
```

---

## Sintaxis de Closures

### Sintaxis Completa

```swift
let greeting: (String) -> String = { (name: String) -> String in
    return "Hello, \(name)!"
}

print(greeting("Ana")) // "Hello, Ana!"
```

### Componentes de un Closure

```swift
{ (parámetros) -> TipoRetorno in
    // cuerpo del closure
    return valor
}
```

### Ejemplo Paso a Paso

```swift
// Array de números
let numbers = [5, 2, 8, 1, 9]

// Forma 1: Sintaxis completa
let sortedComplete = numbers.sorted(by: { (n1: Int, n2: Int) -> Bool in
    return n1 < n2
})

// Forma 2: Inferencia de tipos
let sortedInferred = numbers.sorted(by: { n1, n2 in
    return n1 < n2
})

// Forma 3: Return implícito (expresión única)
let sortedImplicit = numbers.sorted(by: { n1, n2 in n1 < n2 })

// Forma 4: Shorthand argument names
let sortedShorthand = numbers.sorted(by: { $0 < $1 })

// Forma 5: Operator method
let sortedOperator = numbers.sorted(by: <)

print(sortedOperator) // [1, 2, 5, 8, 9]
```

### Closures con Múltiples Parámetros

```swift
// Closure que calcula área de rectángulo
let calculateArea: (Double, Double) -> Double = { width, height in
    width * height
}

print(calculateArea(5, 10)) // 50.0

// Closure sin parámetros
let sayHello: () -> Void = {
    print("Hello!")
}

sayHello() // "Hello!"

// Closure sin retorno pero con parámetros
let printSquare: (Int) -> Void = { number in
    print("Square: \(number * number)")
}

printSquare(5) // "Square: 25"
```

---

## Trailing Closures

Cuando un closure es el último parámetro de una función, puedes usar **trailing closure syntax** para mejorar la legibilidad.

### Sintaxis Básica

```swift
// Función que acepta un closure
func performOperation(operation: () -> Void) {
    print("Starting operation...")
    operation()
    print("Operation completed")
}

// Sin trailing closure
performOperation(operation: {
    print("Doing work...")
})

// Con trailing closure (más limpio)
performOperation {
    print("Doing work...")
}
```

### Ejemplo Realista: Networking

```swift
func fetchData(from url: String, completion: (Data?, Error?) -> Void) {
    // Simular network request
    print("Fetching from \(url)")
    completion(Data(), nil)
}

// Uso con trailing closure
fetchData(from: "https://api.example.com") { data, error in
    if let error = error {
        print("Error: \(error)")
        return
    }
    
    if let data = data {
        print("Data received: \(data)")
    }
}
```

### Múltiples Trailing Closures (Swift 5.3+)

```swift
func loadPicture(from server: String, 
                 completion: (UIImage) -> Void,
                 onFailure: () -> Void) {
    // Implementation
}

// Sintaxis con múltiples trailing closures
loadPicture(from: "server.com") { image in
    print("Loaded: \(image)")
} onFailure: {
    print("Failed to load")
}
```

---

## Capturing Values

Los closures pueden **capturar** valores del contexto en el que fueron definidos.

### Captura Simple

```swift
func makeIncrementer(incrementAmount: Int) -> () -> Int {
    var total = 0
    
    let incrementer: () -> Int = {
        total += incrementAmount  // Captura 'total' y 'incrementAmount'
        return total
    }
    
    return incrementer
}

let incrementByTwo = makeIncrementer(incrementAmount: 2)
print(incrementByTwo()) // 2
print(incrementByTwo()) // 4
print(incrementByTwo()) // 6

let incrementByTen = makeIncrementer(incrementAmount: 10)
print(incrementByTen()) // 10
print(incrementByTen()) // 20
```

### Reference Semantics

```swift
var count = 0

let incrementer = {
    count += 1
    print("Count: \(count)")
}

incrementer() // Count: 1
incrementer() // Count: 2
print(count)  // 2
```

### Captura de Referencias Débiles

```swift
class NetworkManager {
    var data: [String] = []
    
    func fetchData(completion: @escaping () -> Void) {
        // Simular async task
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) { [weak self] in
            guard let self = self else { return }
            self.data.append("New data")
            completion()
        }
    }
}

// Uso
let manager = NetworkManager()
manager.fetchData {
    print("Data fetched")
}
```

### Capture Lists

```swift
class HTMLElement {
    let name: String
    let text: String?
    
    lazy var asHTML: () -> String = { [unowned self] in
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }
    
    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }
}

let heading = HTMLElement(name: "h1", text: "Hello, World!")
print(heading.asHTML()) // "<h1>Hello, World!</h1>"
```

---

## Escaping Closures

Un closure **escaping** es aquel que se ejecuta después de que la función que lo recibe ha retornado.

### @escaping Keyword

```swift
// Closure que NO escapa (se ejecuta dentro de la función)
func performImmediately(operation: () -> Void) {
    operation()
}

// Closure que SÍ escapa (se almacena para usar después)
var completionHandlers: [() -> Void] = []

func addCompletionHandler(handler: @escaping () -> Void) {
    completionHandlers.append(handler)
}

addCompletionHandler {
    print("Handler executed")
}

// Ejecutar todos los handlers después
completionHandlers.forEach { $0() }
```

### Ejemplo con Networking

```swift
class DataManager {
    var completionHandlers: [(Data?) -> Void] = []
    
    func fetchData(completion: @escaping (Data?) -> Void) {
        // Async network call
        URLSession.shared.dataTask(with: URL(string: "https://api.example.com")!) { data, _, error in
            completion(data)
        }.resume()
    }
}

// Uso
let manager = DataManager()
manager.fetchData { data in
    if let data = data {
        print("Received \(data.count) bytes")
    }
}
```

### Escaping con Capture Lists

```swift
class ViewController {
    var name = "ViewController"
    
    func loadData() {
        NetworkService.fetch { [weak self] data in
            guard let self = self else { return }
            print("Data loaded in \(self.name)")
        }
    }
}
```

---

## Autoclosures

Un **autoclosure** es un closure que se crea automáticamente para envolver una expresión.

### @autoclosure Básico

```swift
// Sin autoclosure
func logIfTrue(condition: () -> Bool) {
    if condition() {
        print("Condition is true")
    }
}

logIfTrue(condition: { 2 > 1 }) // Necesitas crear el closure

// Con autoclosure
func logIfTrueAuto(condition: @autoclosure () -> Bool) {
    if condition() {
        print("Condition is true")
    }
}

logIfTrueAuto(condition: 2 > 1) // Más limpio, sin { }
```

### Lazy Evaluation

```swift
// Autoclosure permite evaluación lazy
func assert(_ condition: @autoclosure () -> Bool,
            _ message: @autoclosure () -> String) {
    #if DEBUG
    if !condition() {
        print("Assertion failed: \(message())")
    }
    #endif
}

// message() solo se evalúa si condition() es false
assert(2 > 3, "Two should be greater than three")
```

### Ejemplo de Swift Standard Library

```swift
// Así es como funciona ?? operator internamente
func ?? <T>(optional: T?, defaultValue: @autoclosure () -> T) -> T {
    if let value = optional {
        return value
    } else {
        return defaultValue()
    }
}

var username: String? = nil
let displayName = username ?? "Guest" // "Guest" solo se evalúa si username es nil
```

---

## Map, Filter, Reduce

### Map - Transformar Elementos

```swift
let numbers = [1, 2, 3, 4, 5]

// Duplicar cada número
let doubled = numbers.map { $0 * 2 }
print(doubled) // [2, 4, 6, 8, 10]

// Convertir a Strings
let strings = numbers.map { String($0) }
print(strings) // ["1", "2", "3", "4", "5"]

// Con structs
struct Person {
    let name: String
    let age: Int
}

let people = [
    Person(name: "Ana", age: 25),
    Person(name: "Luis", age: 30),
    Person(name: "María", age: 28)
]

let names = people.map { $0.name }
print(names) // ["Ana", "Luis", "María"]

let ages = people.map { $0.age }
print(ages) // [25, 30, 28]
```

### Filter - Filtrar Elementos

```swift
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

// Solo números pares
let evens = numbers.filter { $0 % 2 == 0 }
print(evens) // [2, 4, 6, 8, 10]

// Solo números mayores que 5
let greaterThanFive = numbers.filter { $0 > 5 }
print(greaterThanFive) // [6, 7, 8, 9, 10]

// Filtrar personas mayores de edad
let adults = people.filter { $0.age >= 18 }

// Combinar map y filter
let evenDoubled = numbers
    .filter { $0 % 2 == 0 }
    .map { $0 * 2 }
print(evenDoubled) // [4, 8, 12, 16, 20]
```

### Reduce - Combinar Elementos

```swift
let numbers = [1, 2, 3, 4, 5]

// Sumar todos los números
let sum = numbers.reduce(0) { $0 + $1 }
// o más simple:
let sumSimple = numbers.reduce(0, +)
print(sum) // 15

// Multiplicar todos
let product = numbers.reduce(1, *)
print(product) // 120

// Concatenar strings
let words = ["Hello", "Swift", "World"]
let sentence = words.reduce("") { $0 + " " + $1 }
print(sentence) // " Hello Swift World"

// Ejemplo complejo: agrupar por propiedad
struct Product {
    let category: String
    let price: Double
}

let products = [
    Product(category: "Electronics", price: 999),
    Product(category: "Books", price: 20),
    Product(category: "Electronics", price: 499)
]

let totalByCategory = products.reduce(into: [:]) { result, product in
    result[product.category, default: 0] += product.price
}
print(totalByCategory) // ["Electronics": 1498.0, "Books": 20.0]
```

### CompactMap - Filtrar Nils

```swift
let possibleNumbers = ["1", "2", "three", "4", "five"]

// map retorna [Int?]
let mapped = possibleNumbers.map { Int($0) }
print(mapped) // [Optional(1), Optional(2), nil, Optional(4), nil]

// compactMap filtra los nil
let compactMapped = possibleNumbers.compactMap { Int($0) }
print(compactMapped) // [1, 2, 4]
```

### FlatMap - Aplanar Arrays

```swift
let arrays = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

// Aplanar array de arrays
let flattened = arrays.flatMap { $0 }
print(flattened) // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// Combinación de operaciones
let result = arrays
    .flatMap { $0 }
    .filter { $0 % 2 == 0 }
    .map { $0 * 2 }
print(result) // [4, 8, 12, 16, 18]
```

---

## Casos de Uso Comunes

### 1. Callbacks en Networking

```swift
class APIClient {
    func fetchUsers(completion: @escaping (Result<[User], Error>) -> Void) {
        URLSession.shared.dataTask(with: URL(string: "https://api.example.com/users")!) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(APIError.noData))
                return
            }
            
            do {
                let users = try JSONDecoder().decode([User].self, from: data)
                completion(.success(users))
            } catch {
                completion(.failure(error))
            }
        }.resume()
    }
}

// Uso
let client = APIClient()
client.fetchUsers { result in
    switch result {
    case .success(let users):
        print("Received \(users.count) users")
    case .failure(let error):
        print("Error: \(error)")
    }
}
```

### 2. Animaciones en UIKit/SwiftUI

```swift
// UIKit
UIView.animate(withDuration: 0.3) {
    self.view.alpha = 0
} completion: { finished in
    if finished {
        self.view.removeFromSuperview()
    }
}

// SwiftUI
withAnimation(.spring()) {
    self.isExpanded.toggle()
}
```

### 3. Sorting y Comparación

```swift
struct Student {
    let name: String
    let grade: Double
}

let students = [
    Student(name: "Ana", grade: 95),
    Student(name: "Luis", grade: 87),
    Student(name: "María", grade: 92)
]

// Ordenar por grade descendente
let sortedByGrade = students.sorted { $0.grade > $1.grade }

// Ordenar por nombre alfabéticamente
let sortedByName = students.sorted { $0.name < $1.name }
```

### 4. Lazy Evaluation

```swift
let numbers = 1...1000

// Lazy evaluation - solo procesa lo necesario
let result = numbers
    .lazy
    .filter { $0 % 2 == 0 }
    .map { $0 * 2 }
    .prefix(5)

print(Array(result)) // [4, 8, 12, 16, 20]
```

### 5. Custom Operators con Closures

```swift
precedencegroup ForwardApplication {
    associativity: left
}

infix operator |>: ForwardApplication

func |> <T, U>(value: T, transform: (T) -> U) -> U {
    transform(value)
}

// Uso
let result = 5
    |> { $0 * 2 }
    |> { $0 + 10 }
    |> { $0 / 2 }

print(result) // 10
```

---

## Checklist de Aprendizaje

### Nivel Básico
- [ ] Entender qué es un closure y por qué son útiles
- [ ] Escribir closures con sintaxis completa
- [ ] Usar trailing closure syntax
- [ ] Aplicar shorthand argument names ($0, $1)
- [ ] Usar closures con funciones comunes (sort, filter, map)

### Nivel Intermedio
- [ ] Entender y aplicar capture lists [weak self], [unowned self]
- [ ] Distinguir entre escaping y non-escaping closures
- [ ] Usar @escaping correctamente
- [ ] Implementar callbacks para operaciones asíncronas
- [ ] Combinar map, filter, reduce en cadenas
- [ ] Usar compactMap para filtrar optionals
- [ ] Aplicar flatMap para aplanar colecciones

### Nivel Avanzado
- [ ] Entender y usar @autoclosure
- [ ] Implementar custom operators con closures
- [ ] Usar lazy evaluation para optimizar performance
- [ ] Crear DSLs (Domain Specific Languages) con closures
- [ ] Manejar memory leaks con weak/unowned references
- [ ] Implementar functional programming patterns

### Práctica
- [ ] Crear una función que use trailing closures
- [ ] Implementar un sistema de validación con closures
- [ ] Crear una API client con callbacks
- [ ] Refactorizar código imperativo a funcional
- [ ] Implementar un parser simple con closures

---

## Próximos Pasos

### 1. Profundizar en Temas Relacionados
- **Protocols** - Combinar closures con protocols
- **Generics** - Closures genéricos
- **Async/Await** - Alternativa moderna a callbacks
- **Combine** - Reactive programming con closures

### 2. Aplicaciones Prácticas
```swift
// Proyecto sugerido: Sistema de Validación
struct Validator<T> {
    let validate: (T) -> Bool
    let errorMessage: String
}

let emailValidator = Validator<String>(
    validate: { $0.contains("@") && $0.contains(".") },
    errorMessage: "Email inválido"
)

func validateInput<T>(_ input: T, with validators: [Validator<T>]) -> [String] {
    validators
        .filter { !$0.validate(input) }
        .map { $0.errorMessage }
}

let errors = validateInput("test", with: [emailValidator])
```

### 3. Patrones Avanzados
- **Builder Pattern** con closures
- **Strategy Pattern** con closures
- **Observer Pattern** con closures
- **Chain of Responsibility** con closures

### 4. Performance y Optimización
- Usar `lazy` para operaciones costosas
- Evitar retain cycles con capture lists
- Optimizar con `inout` parameters cuando sea apropiado

### 5. Recursos para Continuar
- Leer sobre Functional Programming en Swift
- Estudiar el código fuente de librerías populares
- Practicar refactoring de código imperativo a funcional

---

## Recursos

### Documentación Oficial
- 📚 [Closures - Swift Language Guide](https://docs.swift.org/swift-book/LanguageGuide/Closures.html)
- 📖 [Escaping Closures](https://docs.swift.org/swift-book/LanguageGuide/Closures.html#ID546)
- 🎓 [Autoclosures](https://docs.swift.org/swift-book/LanguageGuide/Closures.html#ID543)

### Artículos y Tutoriales
- 📝 [Closures in Swift Explained](https://www.hackingwithswift.com/sixty/6/1/creating-basic-closures) - Paul Hudson
- 📝 [Understanding Closures](https://www.swiftbysundell.com/articles/capturing-objects-in-swift-closures/) - John Sundell
- 📝 [Weak, Strong, Unowned](https://www.avanderlee.com/swift/weak-self/) - Antoine van der Lee
- 📝 [Map, Filter, Reduce](https://useyourloaf.com/blog/swift-guide-to-map-filter-reduce/) - Use Your Loaf

### Videos
- 🎥 [Swift Closures Explained](https://www.youtube.com/watch?v=3cBLKoRYz_k) - Sean Allen
- 🎥 [Escaping Closures](https://www.youtube.com/watch?v=6PYY0VpFTf8) - Paul Hudson
- 🎥 [Map, Filter, Reduce](https://www.youtube.com/watch?v=HgmP8MpFnYE) - CodeWithChris
- 🎥 [Memory Management with Closures](https://www.youtube.com/watch?v=oNMuSh_WShc) - Sean Allen

### Libros
- 📕 "Swift Programming" - Capítulo 13: Closures - Big Nerd Ranch
- 📗 "Functional Swift" - Chris Eidhof
- 📘 "Advanced Swift" - Closures chapter - objc.io
- 📙 "iOS Programming with Swift" - Matt Neuburg

### Cursos
- 🎓 [100 Days of Swift](https://www.hackingwithswift.com/100) - Day 7-9
- 🎓 [Swift Essential Training](https://www.linkedin.com/learning/swift-essential-training) - LinkedIn Learning
- 🎓 [iOS Development Bootcamp](https://www.udemy.com/course/ios-13-app-development-bootcamp/) - Angela Yu

### Práctica Interactiva
- 💻 [Swift Playgrounds](https://www.apple.com/swift/playgrounds/) - Closures exercises
- 💻 [Exercism Swift Track](https://exercism.org/tracks/swift) - Closure exercises
- 💻 [HackerRank Swift](https://www.hackerrank.com/domains/swift)
- 💻 [Codewars Swift](https://www.codewars.com/?language=swift)

### Herramientas
- 🛠️ [SwiftLint](https://github.com/realm/SwiftLint) - Detecta retain cycles
- 🛠️ [Instruments](https://developer.apple.com/instruments/) - Memory profiling
- 🛠️ [Xcode Memory Debugger](https://developer.apple.com/documentation/xcode/debugging-memory-issues)

### Comunidad
- 💬 [Swift Forums - Closures](https://forums.swift.org)
- 💬 [r/swift](https://reddit.com/r/swift)
- 💬 [Stack Overflow Swift Closures](https://stackoverflow.com/questions/tagged/swift+closures)

---

**Anterior:** [Optionals](Optionals.md)  
**Próximo:** [Protocols & Extensions](Protocols-Extensions.md)
