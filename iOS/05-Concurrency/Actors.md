# Actors - Thread Safety

## ¿Qué son los Actors?

Actors protegen el estado mutable de race conditions.

```swift
actor Counter {
    private var value = 0
    
    func increment() {
        value += 1
    }
    
    func getValue() -> Int {
        value
    }
}

// Uso
let counter = Counter()

Task {
    await counter.increment()
    let value = await counter.getValue()
    print(value) // 1
}
```

## Actor con Async Functions

```swift
actor ImageCache {
    private var cache: [URL: UIImage] = [:]
    
    func image(for url: URL) async -> UIImage? {
        if let cached = cache[url] {
            return cached
        }
        
        guard let (data, _) = try? await URLSession.shared.data(from: url),
              let image = UIImage(data: data) else {
            return nil
        }
        
        cache[url] = image
        return image
    }
}
```

## @MainActor

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var data: [Item] = []
    
    func loadData() async {
        // Automáticamente en main thread
        data = await fetchData()
    }
}
```

## Recursos

- 📚 [Actors](https://docs.swift.org/swift-book/LanguageGuide/Concurrency.html#ID645)
- 🎥 [Protect mutable state with Swift actors](https://developer.apple.com/videos/play/wwdc2021/10133/)
