# Navigation en SwiftUI

## NavigationStack

```swift
struct ContentView: View {
    var body: some View {
        NavigationStack {
            List(items) { item in
                NavigationLink(item.name, value: item)
            }
            .navigationDestination(for: Item.self) { item in
                DetailView(item: item)
            }
            .navigationTitle("Items")
        }
    }
}
```

## NavigationLink

```swift
NavigationLink("Go to Detail") {
    DetailView()
}
```

## Programmatic Navigation

```swift
@State private var path = NavigationPath()

NavigationStack(path: $path) {
    // ...
}

path.append(item) // Navigate
path.removeLast() // Go back
```

## Recursos
- [Navigation](https://developer.apple.com/documentation/swiftui/navigation)
