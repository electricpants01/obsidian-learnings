# Singleton Pattern

## Implementación

```swift
class NetworkManager {
    static let shared = NetworkManager()
    
    private init() {
        // Private init prevents external instantiation
    }
    
    func fetchData() {
        print("Fetching data...")
    }
}

// Uso
NetworkManager.shared.fetchData()
```

## Recursos
- [Design Patterns](https://refactoring.guru/design-patterns/swift)
