# Publishers & Subscribers

## Publisher

```swift
import Combine

let publisher = Just("Hello, Combine!")
let publisher2 = [1, 2, 3].publisher
```

## Subscriber

```swift
var cancellables = Set<AnyCancellable>()

publisher
    .sink { value in
        print(value)
    }
    .store(in: &cancellables)
```

## @Published

```swift
class ViewModel: ObservableObject {
    @Published var username = ""
    @Published var isValid = false
    
    init() {
        $username
            .map { $0.count >= 3 }
            .assign(to: &$isValid)
    }
}
```

## Recursos
- [Combine](https://developer.apple.com/documentation/combine)
