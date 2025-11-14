# Navigation en SwiftUI

## NavigationStack (iOS 16+)

```swift
struct ContentView: View {
    var body: some View {
        NavigationStack {
            List(1...10, id: \.self) { number in
                NavigationLink("Item \(number)", value: number)
            }
            .navigationDestination(for: Int.self) { number in
                DetailView(number: number)
            }
            .navigationTitle("List")
        }
    }
}

struct DetailView: View {
    let number: Int
    
    var body: some View {
        Text("Detail \(number)")
            .navigationTitle("Detail")
    }
}
```

## NavigationPath

```swift
@Observable
class NavigationManager {
    var path = NavigationPath()
    
    func popToRoot() {
        path = NavigationPath()
    }
    
    func pop() {
        if !path.isEmpty {
            path.removeLast()
        }
    }
}

struct AppView: View {
    @State private var manager = NavigationManager()
    
    var body: some View {
        NavigationStack(path: $manager.path) {
            // Content
        }
    }
}
```

## Sheet Presentation

```swift
struct ContentView: View {
    @State private var showSheet = false
    
    var body: some View {
        Button("Show Sheet") {
            showSheet = true
        }
        .sheet(isPresented: $showSheet) {
            SheetView()
        }
    }
}
```

## FullScreenCover

```swift
struct ContentView: View {
    @State private var showFullScreen = false
    
    var body: some View {
        Button("Show Full Screen") {
            showFullScreen = true
        }
        .fullScreenCover(isPresented: $showFullScreen) {
            FullScreenView()
        }
    }
}
```

## Recursos

- 📚 [NavigationStack](https://developer.apple.com/documentation/swiftui/navigationstack)
- 🎥 [Navigation in SwiftUI](https://www.youtube.com/watch?v=4GjXq2Sr55Q)
