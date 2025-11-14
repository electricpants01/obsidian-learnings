# Animations en SwiftUI

## Animaciones Básicas

```swift
struct AnimatedView: View {
    @State private var scale = 1.0
    
    var body: some View {
        Circle()
            .scaleEffect(scale)
            .onTapGesture {
                withAnimation {
                    scale = scale == 1.0 ? 1.5 : 1.0
                }
            }
    }
}
```

## Tipos de Animaciones

```swift
// Linear
withAnimation(.linear(duration: 1.0)) {
    offset = 100
}

// EaseIn/EaseOut
withAnimation(.easeInOut) {
    rotation += 180
}

// Spring
withAnimation(.spring(response: 0.5, dampingFraction: 0.6)) {
    position.y = 200
}
```

## Transitions

```swift
struct TransitionView: View {
    @State private var showDetails = false
    
    var body: some View {
        VStack {
            Button("Toggle") {
                withAnimation {
                    showDetails.toggle()
                }
            }
            
            if showDetails {
                Text("Details")
                    .transition(.slide)
            }
        }
    }
}
```

## Animaciones Personalizadas

```swift
struct ShakeEffect: GeometryEffect {
    var amount: Double = 10
    var shakesPerUnit = 3
    var animatableData: Double
    
    func effectValue(size: CGSize) -> ProjectionTransform {
        ProjectionTransform(CGAffineTransform(
            translationX: amount * sin(animatableData * .pi * Double(shakesPerUnit)),
            y: 0
        ))
    }
}

// Uso
Text("Shake Me")
    .modifier(ShakeEffect(animatableData: shakeValue))
```

## Recursos

- 📚 [Animations](https://developer.apple.com/documentation/swiftui/animation)
- 🎥 [SwiftUI Animations](https://www.youtube.com/watch?v=3krC2c56ceQ)
