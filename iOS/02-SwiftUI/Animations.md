# Animations en SwiftUI - Guía Completa

## Tabla de Contenido
1. [Introducción a Animaciones](#introducción-a-animaciones)
2. [Animaciones Básicas](#animaciones-básicas)
3. [Tipos de Animaciones](#tipos-de-animaciones)
4. [Transitions](#transitions)
5. [Animaciones Personalizadas](#animaciones-personalizadas)
6. [GeometryEffect](#geometryeffect)
7. [Animation Modifiers](#animation-modifiers)
8. [Spring Animations](#spring-animations)
9. [Keyframe Animations](#keyframe-animations)
10. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
11. [Próximos Pasos](#próximos-pasos)
12. [Recursos](#recursos)

---

## Introducción a Animaciones

SwiftUI hace que las animaciones sean simples y declarativas. Cualquier cambio de estado puede ser animado.

### Concepto Básico

```swift
struct BasicAnimationConcept: View {
    @State private var isExpanded = false
    
    var body: some View {
        VStack {
            // Sin animación
            Rectangle()
                .fill(Color.blue)
                .frame(width: isExpanded ? 200 : 100, height: 100)
            
            // Con animación
            Rectangle()
                .fill(Color.green)
                .frame(width: isExpanded ? 200 : 100, height: 100)
                .animation(.default, value: isExpanded)
            
            Button("Toggle") {
                isExpanded.toggle()
            }
        }
    }
}
```

---

## Animaciones Básicas

### withAnimation

```swift
struct WithAnimationExample: View {
    @State private var scale: CGFloat = 1.0
    @State private var rotation: Double = 0
    @State private var opacity: Double = 1.0
    
    var body: some View {
        VStack(spacing: 30) {
            Circle()
                .fill(Color.blue)
                .frame(width: 100 * scale, height: 100 * scale)
                .rotationEffect(.degrees(rotation))
                .opacity(opacity)
            
            HStack(spacing: 20) {
                Button("Scale") {
                    withAnimation {
                        scale = scale == 1.0 ? 1.5 : 1.0
                    }
                }
                
                Button("Rotate") {
                    withAnimation {
                        rotation += 180
                    }
                }
                
                Button("Fade") {
                    withAnimation {
                        opacity = opacity == 1.0 ? 0.3 : 1.0
                    }
                }
            }
        }
    }
}
```

### Explicit Animations

```swift
struct ExplicitAnimations: View {
    @State private var position: CGFloat = 0
    
    var body: some View {
        VStack {
            Circle()
                .fill(Color.red)
                .frame(width: 50, height: 50)
                .offset(x: position)
            
            Button("Move") {
                withAnimation(.easeInOut(duration: 1.0)) {
                    position = position == 0 ? 200 : 0
                }
            }
        }
    }
}
```

### Implicit Animations

```swift
struct ImplicitAnimations: View {
    @State private var scale = 1.0
    
    var body: some View {
        Circle()
            .fill(Color.purple)
            .frame(width: 100 * scale, height: 100 * scale)
            .animation(.easeInOut, value: scale)
            .onTapGesture {
                scale = scale == 1.0 ? 1.5 : 1.0
            }
    }
}
```

---

## Tipos de Animaciones

### Linear

```swift
struct LinearAnimation: View {
    @State private var offset: CGFloat = 0
    
    var body: some View {
        VStack {
            Circle()
                .fill(Color.blue)
                .frame(width: 50, height: 50)
                .offset(y: offset)
            
            Button("Animate") {
                withAnimation(.linear(duration: 2.0)) {
                    offset = offset == 0 ? 200 : 0
                }
            }
        }
    }
}
```

### EaseIn, EaseOut, EaseInOut

```swift
struct EasingAnimations: View {
    @State private var offset1: CGFloat = 0
    @State private var offset2: CGFloat = 0
    @State private var offset3: CGFloat = 0
    
    var body: some View {
        VStack(spacing: 40) {
            HStack {
                Text("EaseIn")
                Spacer()
                Circle()
                    .fill(Color.red)
                    .frame(width: 30, height: 30)
                    .offset(x: offset1)
            }
            
            HStack {
                Text("EaseOut")
                Spacer()
                Circle()
                    .fill(Color.green)
                    .frame(width: 30, height: 30)
                    .offset(x: offset2)
            }
            
            HStack {
                Text("EaseInOut")
                Spacer()
                Circle()
                    .fill(Color.blue)
                    .frame(width: 30, height: 30)
                    .offset(x: offset3)
            }
            
            Button("Animate All") {
                withAnimation(.easeIn(duration: 1.5)) {
                    offset1 = offset1 == 0 ? 200 : 0
                }
                withAnimation(.easeOut(duration: 1.5)) {
                    offset2 = offset2 == 0 ? 200 : 0
                }
                withAnimation(.easeInOut(duration: 1.5)) {
                    offset3 = offset3 == 0 ? 200 : 0
                }
            }
        }
        .padding()
    }
}
```

---

## Transitions

### Basic Transitions

```swift
struct BasicTransitions: View {
    @State private var showDetails = false
    
    var body: some View {
        VStack(spacing: 20) {
            Button("Toggle Details") {
                withAnimation {
                    showDetails.toggle()
                }
            }
            
            if showDetails {
                Text("Details appear here")
                    .transition(.slide)
            }
        }
    }
}
```

### Available Transitions

```swift
struct TransitionTypes: View {
    @State private var show1 = false
    @State private var show2 = false
    @State private var show3 = false
    @State private var show4 = false
    
    var body: some View {
        VStack(spacing: 20) {
            Button("Slide") {
                withAnimation {
                    show1.toggle()
                }
            }
            if show1 {
                Text("Slide")
                    .transition(.slide)
            }
            
            Button("Scale") {
                withAnimation {
                    show2.toggle()
                }
            }
            if show2 {
                Text("Scale")
                    .transition(.scale)
            }
            
            Button("Opacity") {
                withAnimation {
                    show3.toggle()
                }
            }
            if show3 {
                Text("Opacity")
                    .transition(.opacity)
            }
            
            Button("Move") {
                withAnimation {
                    show4.toggle()
                }
            }
            if show4 {
                Text("Move")
                    .transition(.move(edge: .bottom))
            }
        }
    }
}
```

### Combined Transitions

```swift
struct CombinedTransitions: View {
    @State private var showCard = false
    
    var body: some View {
        VStack {
            Button("Toggle Card") {
                withAnimation(.spring(response: 0.6, dampingFraction: 0.8)) {
                    showCard.toggle()
                }
            }
            
            if showCard {
                RoundedRectangle(cornerRadius: 20)
                    .fill(Color.blue)
                    .frame(width: 200, height: 300)
                    .transition(.scale.combined(with: .opacity))
            }
        }
    }
}
```

### Asymmetric Transitions

```swift
struct AsymmetricTransitions: View {
    @State private var show = false
    
    var body: some View {
        VStack {
            Button("Toggle") {
                withAnimation {
                    show.toggle()
                }
            }
            
            if show {
                Rectangle()
                    .fill(Color.green)
                    .frame(width: 100, height: 100)
                    .transition(.asymmetric(
                        insertion: .scale,
                        removal: .slide
                    ))
            }
        }
    }
}
```

---

## Animaciones Personalizadas

### Custom Animation

```swift
struct CustomAnimation: View {
    @State private var animate = false
    
    var body: some View {
        VStack {
            Circle()
                .fill(Color.purple)
                .frame(width: 100, height: 100)
                .scaleEffect(animate ? 1.5 : 1.0)
                .rotationEffect(.degrees(animate ? 360 : 0))
                .animation(
                    Animation.easeInOut(duration: 2.0)
                        .repeatCount(3, autoreverses: true),
                    value: animate
                )
            
            Button("Animate") {
                animate.toggle()
            }
        }
    }
}
```

### Delay y Speed

```swift
struct DelayedAnimation: View {
    @State private var animate = false
    
    var body: some View {
        HStack(spacing: 20) {
            ForEach(0..<5) { index in
                Circle()
                    .fill(Color.blue)
                    .frame(width: 30, height: 30)
                    .offset(y: animate ? -50 : 0)
                    .animation(
                        Animation.easeInOut(duration: 0.5)
                            .delay(Double(index) * 0.1)
                            .repeatForever(autoreverses: true),
                        value: animate
                    )
            }
        }
        .onAppear {
            animate = true
        }
    }
}
```

### Repeat Animations

```swift
struct RepeatAnimation: View {
    @State private var pulse = false
    
    var body: some View {
        Circle()
            .fill(Color.red)
            .frame(width: 100, height: 100)
            .scaleEffect(pulse ? 1.2 : 1.0)
            .animation(
                Animation.easeInOut(duration: 1.0)
                    .repeatForever(autoreverses: true),
                value: pulse
            )
            .onAppear {
                pulse = true
            }
    }
}
```

---

## GeometryEffect

### Shake Effect

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

struct ShakeView: View {
    @State private var attempts = 0
    
    var body: some View {
        VStack {
            TextField("Password", text: .constant(""))
                .textFieldStyle(.roundedBorder)
                .modifier(ShakeEffect(animatableData: Double(attempts)))
            
            Button("Wrong Password") {
                withAnimation(.default) {
                    attempts += 1
                }
            }
        }
        .padding()
    }
}
```

### Wave Effect

```swift
struct WaveEffect: GeometryEffect {
    var time: Double
    var amplitude: Double = 10
    var wavelength: Double = 50
    
    var animatableData: Double {
        get { time }
        set { time = newValue }
    }
    
    func effectValue(size: CGSize) -> ProjectionTransform {
        let yOffset = amplitude * sin((time * 2 * .pi) / wavelength)
        return ProjectionTransform(CGAffineTransform(translationX: 0, y: yOffset))
    }
}

struct WaveView: View {
    @State private var time = 0.0
    
    var body: some View {
        Text("Wave Effect")
            .font(.title)
            .modifier(WaveEffect(time: time))
            .onAppear {
                withAnimation(.linear(duration: 2.0).repeatForever(autoreverses: false)) {
                    time = 100
                }
            }
    }
}
```

---

## Animation Modifiers

### Animation Curves

```swift
struct AnimationCurves: View {
    @State private var move = false
    
    var body: some View {
        VStack(spacing: 40) {
            Text("Default")
            Circle()
                .frame(width: 30, height: 30)
                .offset(x: move ? 150 : -150)
                .animation(.default, value: move)
            
            Text("EaseIn")
            Circle()
                .frame(width: 30, height: 30)
                .offset(x: move ? 150 : -150)
                .animation(.easeIn, value: move)
            
            Text("EaseOut")
            Circle()
                .frame(width: 30, height: 30)
                .offset(x: move ? 150 : -150)
                .animation(.easeOut, value: move)
            
            Button("Animate") {
                move.toggle()
            }
        }
    }
}
```

### Timing Curves

```swift
struct TimingCurves: View {
    @State private var animate = false
    
    var body: some View {
        VStack(spacing: 30) {
            Circle()
                .fill(Color.blue)
                .frame(width: 50, height: 50)
                .offset(x: animate ? 150 : -150)
                .animation(.timingCurve(0.5, 0, 0.5, 1, duration: 1.0), value: animate)
            
            Button("Animate") {
                animate.toggle()
            }
        }
    }
}
```

---

## Spring Animations

### Basic Spring

```swift
struct SpringAnimation: View {
    @State private var scale: CGFloat = 1.0
    
    var body: some View {
        VStack {
            Circle()
                .fill(Color.orange)
                .frame(width: 100 * scale, height: 100 * scale)
            
            Button("Bounce") {
                withAnimation(.spring(response: 0.5, dampingFraction: 0.5)) {
                    scale = scale == 1.0 ? 1.5 : 1.0
                }
            }
        }
    }
}
```

### Custom Spring

```swift
struct CustomSpring: View {
    @State private var position: CGFloat = 0
    
    var body: some View {
        VStack(spacing: 40) {
            Circle()
                .fill(Color.green)
                .frame(width: 50, height: 50)
                .offset(y: position)
            
            VStack {
                Button("Bouncy") {
                    withAnimation(.spring(response: 0.3, dampingFraction: 0.3)) {
                        position = position == 0 ? 200 : 0
                    }
                }
                
                Button("Smooth") {
                    withAnimation(.spring(response: 0.5, dampingFraction: 0.8)) {
                        position = position == 0 ? 200 : 0
                    }
                }
                
                Button("Stiff") {
                    withAnimation(.spring(response: 0.1, dampingFraction: 0.7)) {
                        position = position == 0 ? 200 : 0
                    }
                }
            }
        }
    }
}
```

---

## Keyframe Animations

### Keyframe Animation (iOS 17+)

```swift
struct KeyframeAnimation: View {
    @State private var animate = false
    
    var body: some View {
        Circle()
            .fill(Color.blue)
            .frame(width: 100, height: 100)
            .keyframeAnimator(initialValue: AnimationValues(), trigger: animate) { content, value in
                content
                    .scaleEffect(value.scale)
                    .rotationEffect(.degrees(value.rotation))
                    .offset(y: value.verticalOffset)
            } keyframes: { _ in
                KeyframeTrack(\.scale) {
                    SpringKeyframe(1.5, duration: 0.3)
                    SpringKeyframe(1.0, duration: 0.3)
                }
                
                KeyframeTrack(\.rotation) {
                    LinearKeyframe(360, duration: 0.6)
                }
                
                KeyframeTrack(\.verticalOffset) {
                    SpringKeyframe(-50, duration: 0.3)
                    SpringKeyframe(0, duration: 0.3)
                }
            }
            .onTapGesture {
                animate.toggle()
            }
    }
}

struct AnimationValues {
    var scale = 1.0
    var rotation = 0.0
    var verticalOffset = 0.0
}
```

---

## Checklist de Aprendizaje

### Básico
- [ ] Usar withAnimation
- [ ] Aplicar transitions básicas
- [ ] Animaciones implícitas vs explícitas
- [ ] Timing curves básicos

### Intermedio
- [ ] Custom transitions
- [ ] Spring animations
- [ ] Animation delays
- [ ] Repeat animations
- [ ] Combined transitions

### Avanzado
- [ ] GeometryEffect custom
- [ ] Keyframe animations
- [ ] Complex animation sequences
- [ ] Performance optimization
- [ ] Matched geometry effect

---

## Próximos Pasos

### 1. Animaciones Avanzadas
```swift
// Matched Geometry Effect
@Namespace private var animation

struct Hero: View {
    @State private var show = false
    
    var body: some View {
        if !show {
            Circle()
                .matchedGeometryEffect(id: "shape", in: animation)
                .frame(width: 100, height: 100)
        } else {
            Rectangle()
                .matchedGeometryEffect(id: "shape", in: animation)
                .frame(width: 200, height: 200)
        }
    }
}
```

### 2. Performance
- Minimize animated properties
- Use drawingGroup() for complex animations
- Profile con Instruments

---

## Recursos

### Documentación
- 📚 [Animations](https://developer.apple.com/documentation/swiftui/animation)
- 📖 [Transitions](https://developer.apple.com/documentation/swiftui/anytransition)

### Videos
- 🎥 [SwiftUI Animations](https://www.youtube.com/watch?v=3krC2c56ceQ)
- 🎥 [Advanced Animations](https://developer.apple.com/videos/play/wwdc2023/10156/)

### Artículos
- 📝 [Hacking with Swift - Animations](https://www.hackingwithswift.com/books/ios-swiftui/animation-wrap-up)
- 📝 [Swift by Sundell](https://www.swiftbysundell.com/articles/building-smooth-animations-in-swiftui/)

---

**Anterior:** [Lists & Forms](Lists-Forms.md)  
**Próximo:** [MVVM](../04-Arquitectura/MVVM.md)
