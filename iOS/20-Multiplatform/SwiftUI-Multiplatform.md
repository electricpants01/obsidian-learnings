# SwiftUI Multiplataforma

## Código Compartido

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Text("Works on iOS, macOS, watchOS, tvOS")
            #if os(iOS)
            Text("iOS specific")
            #elseif os(macOS)
            Text("macOS specific")
            #endif
        }
    }
}
```

## Recursos
- [Multiplatform](https://developer.apple.com/documentation/swiftui/building-a-multiplatform-app)
