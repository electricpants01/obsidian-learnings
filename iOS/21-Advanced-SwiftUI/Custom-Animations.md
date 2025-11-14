# Custom Animations

## Spring Animation

```swift
struct AnimatedView: View {
    @State private var scale = 1.0
    
    var body: some View {
        Circle()
            .scaleEffect(scale)
            .onTapGesture {
                withAnimation(.spring(response: 0.5, dampingFraction: 0.6)) {
                    scale = scale == 1.0 ? 1.5 : 1.0
                }
            }
    }
}
```

## Recursos
- [Animations](https://developer.apple.com/documentation/swiftui/animation)
