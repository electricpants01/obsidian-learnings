# Views & Modifiers en SwiftUI

## Basic Views

```swift
// Text
Text("Hello, World!")
    .font(.title)
    .foregroundColor(.blue)
    .padding()

// Image
Image(systemName: "star.fill")
    .resizable()
    .frame(width: 50, height: 50)
    .foregroundColor(.yellow)

// Button
Button("Tap Me") {
    print("Tapped")
}
.buttonStyle(.borderedProminent)

// TextField
@State private var text = ""

TextField("Enter text", text: $text)
    .textFieldStyle(.roundedBorder)
    .padding()
```

## Layout Views

```swift
// VStack
VStack(spacing: 20) {
    Text("Top")
    Text("Middle")
    Text("Bottom")
}

// HStack
HStack(alignment: .center, spacing: 10) {
    Image(systemName: "star")
    Text("Rating")
}

// ZStack
ZStack {
    Color.blue
    Text("Overlay")
        .foregroundColor(.white)
}

// LazyVStack para performance
ScrollView {
    LazyVStack {
        ForEach(1...1000, id: \.self) { number in
            Text("Row \(number)")
        }
    }
}
```

## Modifiers Comunes

```swift
Text("Styled Text")
    .font(.headline)
    .foregroundColor(.primary)
    .padding()
    .background(Color.gray.opacity(0.2))
    .cornerRadius(10)
    .shadow(radius: 5)
    .frame(maxWidth: .infinity)
```

## Custom Modifiers

```swift
struct CardModifier: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding()
            .background(Color.white)
            .cornerRadius(10)
            .shadow(radius: 3)
    }
}

extension View {
    func cardStyle() -> some View {
        modifier(CardModifier())
    }
}

// Uso
Text("Card Content")
    .cardStyle()
```

## Recursos

- 📚 [SwiftUI Views](https://developer.apple.com/documentation/swiftui/views-and-controls)
- 🎥 [SwiftUI Views Tutorial](https://www.youtube.com/watch?v=3krC2c56ceQ)
