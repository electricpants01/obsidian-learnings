# Publishers & Subscribers en Combine

## Publishers

```swift
import Combine

// Just - Emite un valor
let publisher = Just("Hello, Combine!")

// Array Publisher
let numbersPublisher = [1, 2, 3, 4, 5].publisher

// Future - Async operation
let futurePublisher = Future<String, Error> { promise in
    DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
        promise(.success("Completed"))
    }
}

// PassthroughSubject - Emite valores manualmente
let subject = PassthroughSubject<String, Never>()
subject.send("Value 1")
subject.send("Value 2")

// CurrentValueSubject - Mantiene valor actual
let currentSubject = CurrentValueSubject<Int, Never>(0)
print(currentSubject.value) // 0
currentSubject.send(1)
print(currentSubject.value) // 1
```

## Subscribers

```swift
var cancellables = Set<AnyCancellable>()

// sink - Recibe valores
publisher
    .sink { value in
        print(value)
    }
    .store(in: &cancellables)

// sink con completion
publisher
    .sink(
        receiveCompletion: { completion in
            switch completion {
            case .finished:
                print("Finished")
            case .failure(let error):
                print("Error: \(error)")
            }
        },
        receiveValue: { value in
            print("Value: \(value)")
        }
    )
    .store(in: &cancellables)

// assign - Asigna valores a propiedad
class ViewModel: ObservableObject {
    @Published var text = ""
}

let viewModel = ViewModel()

Just("New text")
    .assign(to: \.text, on: viewModel)
    .store(in: &cancellables)
```

## @Published

```swift
class UserViewModel: ObservableObject {
    @Published var username = ""
    @Published var isValid = false
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        $username
            .map { $0.count >= 3 }
            .assign(to: &$isValid)
    }
}
```

## Operators

```swift
// map
[1, 2, 3].publisher
    .map { $0 * 2 }
    .sink { print($0) } // 2, 4, 6

// filter
[1, 2, 3, 4, 5].publisher
    .filter { $0 % 2 == 0 }
    .sink { print($0) } // 2, 4

// combineLatest
let publisher1 = PassthroughSubject<String, Never>()
let publisher2 = PassthroughSubject<Int, Never>()

publisher1
    .combineLatest(publisher2)
    .sink { text, number in
        print("\(text): \(number)")
    }
    .store(in: &cancellables)

// debounce - Espera antes de emitir
searchTextField.publisher
    .debounce(for: .milliseconds(300), scheduler: RunLoop.main)
    .sink { query in
        // Perform search
    }
    .store(in: &cancellables)
```

## Networking con Combine

```swift
struct User: Codable {
    let id: Int
    let name: String
}

func fetchUsers() -> AnyPublisher<[User], Error> {
    let url = URL(string: "https://api.example.com/users")!
    
    return URLSession.shared
        .dataTaskPublisher(for: url)
        .map(\.data)
        .decode(type: [User].self, decoder: JSONDecoder())
        .eraseToAnyPublisher()
}

// Uso
fetchUsers()
    .receive(on: DispatchQueue.main)
    .sink(
        receiveCompletion: { completion in
            if case .failure(let error) = completion {
                print("Error: \(error)")
            }
        },
        receiveValue: { users in
            print("Users: \(users)")
        }
    )
    .store(in: &cancellables)
```

## Recursos

- 📚 [Combine](https://developer.apple.com/documentation/combine)
- 🎥 [Combine Tutorial](https://www.youtube.com/watch?v=3bYtfJ4zvT4)
- 📝 [Using Combine](https://www.hackingwithswift.com/books/ios-swiftui/using-combine)
