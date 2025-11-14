# Views & Modifiers en SwiftUI - Guía Completa

## Tabla de Contenido
1. [Text Views](#text-views)
2. [Image Views](#image-views)
3. [Buttons](#buttons)
4. [TextFields y Forms](#textfields-y-forms)
5. [Layout Containers](#layout-containers)
6. [Stacks](#stacks)
7. [ScrollView y LazyStacks](#scrollview-y-lazystacks)
8. [Modifiers Comunes](#modifiers-comunes)
9. [Custom Modifiers](#custom-modifiers)
10. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
11. [Próximos Pasos](#próximos-pasos)
12. [Recursos](#recursos)

---

## Text Views

### Text Básico

```swift
struct TextExamples: View {
    var body: some View {
        VStack(spacing: 20) {
            // Básico
            Text("Hello, SwiftUI!")
            
            // Font
            Text("Title").font(.title)
            Text("Headline").font(.headline)
            Text("Body").font(.body)
            Text("Caption").font(.caption)
            
            // Custom Font
            Text("Custom Size")
                .font(.system(size: 24, weight: .bold, design: .rounded))
            
            // Color
            Text("Colored Text")
                .foregroundColor(.blue)
            
            // Multiple modifiers
            Text("Styled Text")
                .font(.title2)
                .fontWeight(.semibold)
                .foregroundColor(.purple)
                .italic()
                .underline()
                .strikethrough()
        }
    }
}
```

### Text Formatting

```swift
struct FormattedText: View {
    let price = 99.99
    let date = Date()
    
    var body: some View {
        VStack(alignment: .leading, spacing: 15) {
            // String interpolation
            Text("Price: $\(price, specifier: "%.2f")")
            
            // Date formatting
            Text(date, style: .date)
            Text(date, style: .time)
            Text(date, style: .relative)
            
            // Number formatting
            Text(12345, format: .number)
            Text(0.85, format: .percent)
            Text(1234.56, format: .currency(code: "USD"))
        }
    }
}
```

### Attributed Text

```swift
struct AttributedTextView: View {
    var body: some View {
        VStack(spacing: 20) {
            // Concatenación
            Text("Hello, ")
                .font(.body)
            + Text("World!")
                .font(.title)
                .foregroundColor(.blue)
            
            // Markdown (iOS 15+)
            Text("**Bold** and *Italic* text")
            Text("[Link](https://apple.com)")
        }
    }
}
```

---

## Image Views

### SF Symbols

```swift
struct ImageExamples: View {
    var body: some View {
        VStack(spacing: 20) {
            // SF Symbol básico
            Image(systemName: "star.fill")
            
            // Con tamaño
            Image(systemName: "heart.fill")
                .font(.system(size: 50))
            
            // Con color
            Image(systemName: "bell.fill")
                .foregroundColor(.red)
                .font(.largeTitle)
            
            // Resizable
            Image(systemName: "cloud.sun.fill")
                .resizable()
                .scaledToFit()
                .frame(width: 100, height: 100)
            
            // Symbol variants (iOS 15+)
            Image(systemName: "bell")
                .symbolVariant(.fill)
                .symbolRenderingMode(.palette)
                .foregroundStyle(.blue, .green)
        }
    }
}
```

### Asset Images

```swift
struct AssetImages: View {
    var body: some View {
        VStack {
            // Imagen de Assets
            Image("myImage")
                .resizable()
                .scaledToFit()
                .frame(width: 200, height: 200)
            
            // Aspect ratio
            Image("myImage")
                .resizable()
                .aspectRatio(contentMode: .fill)
                .frame(width: 150, height: 150)
                .clipped()
            
            // Corner radius
            Image("myImage")
                .resizable()
                .scaledToFill()
                .frame(width: 100, height: 100)
                .cornerRadius(10)
        }
    }
}
```

### AsyncImage (iOS 15+)

```swift
struct AsyncImageView: View {
    let url = URL(string: "https://example.com/image.jpg")!
    
    var body: some View {
        VStack {
            // Básico
            AsyncImage(url: url)
            
            // Con placeholder
            AsyncImage(url: url) { image in
                image
                    .resizable()
                    .scaledToFit()
            } placeholder: {
                ProgressView()
            }
            .frame(width: 200, height: 200)
            
            // Con phases
            AsyncImage(url: url) { phase in
                switch phase {
                case .empty:
                    ProgressView()
                case .success(let image):
                    image
                        .resizable()
                        .scaledToFit()
                case .failure:
                    Image(systemName: "exclamationmark.triangle")
                @unknown default:
                    EmptyView()
                }
            }
            .frame(width: 200, height: 200)
        }
    }
}
```

---

## Buttons

### Button Styles

```swift
struct ButtonExamples: View {
    var body: some View {
        VStack(spacing: 15) {
            // Básico
            Button("Tap Me") {
                print("Tapped")
            }
            
            // Con action
            Button(action: {
                print("Button tapped")
            }) {
                Text("Custom Label")
                    .padding()
                    .background(Color.blue)
                    .foregroundColor(.white)
                    .cornerRadius(8)
            }
            
            // Built-in styles (iOS 15+)
            Button("Automatic") { }
                .buttonStyle(.automatic)
            
            Button("Bordered") { }
                .buttonStyle(.bordered)
            
            Button("Bordered Prominent") { }
                .buttonStyle(.borderedProminent)
            
            Button("Plain") { }
                .buttonStyle(.plain)
            
            // Con icono
            Button {
                // Action
            } label: {
                Label("Download", systemImage: "arrow.down.circle.fill")
            }
            .buttonStyle(.borderedProminent)
            
            // Tint
            Button("Colored") { }
                .buttonStyle(.borderedProminent)
                .tint(.purple)
        }
    }
}
```

### Custom Button Styles

```swift
struct CustomButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .padding()
            .background(configuration.isPressed ? Color.gray : Color.blue)
            .foregroundColor(.white)
            .cornerRadius(10)
            .scaleEffect(configuration.isPressed ? 0.95 : 1.0)
    }
}

struct CustomButtonExample: View {
    var body: some View {
        Button("Custom Style") {
            print("Tapped")
        }
        .buttonStyle(CustomButtonStyle())
    }
}
```

---

## TextFields y Forms

### TextField

```swift
struct TextFieldExamples: View {
    @State private var name = ""
    @State private var email = ""
    @State private var password = ""
    
    var body: some View {
        VStack(spacing: 20) {
            // Básico
            TextField("Name", text: $name)
                .textFieldStyle(.roundedBorder)
            
            // Con placeholder
            TextField("Email", text: $email)
                .textFieldStyle(.roundedBorder)
                .keyboardType(.emailAddress)
                .textContentType(.emailAddress)
                .autocapitalization(.none)
            
            // SecureField
            SecureField("Password", text: $password)
                .textFieldStyle(.roundedBorder)
            
            // Con validación visual
            TextField("Email", text: $email)
                .textFieldStyle(.roundedBorder)
                .overlay(
                    RoundedRectangle(cornerRadius: 5)
                        .stroke(email.contains("@") ? Color.green : Color.red, lineWidth: 1)
                )
        }
        .padding()
    }
}
```

### Picker

```swift
struct PickerExamples: View {
    @State private var selectedFruit = "Apple"
    let fruits = ["Apple", "Banana", "Orange", "Grape"]
    
    @State private var selectedIndex = 0
    
    var body: some View {
        Form {
            // Picker básico
            Picker("Fruit", selection: $selectedFruit) {
                ForEach(fruits, id: \.self) { fruit in
                    Text(fruit).tag(fruit)
                }
            }
            
            // Picker con índice
            Picker("Select", selection: $selectedIndex) {
                Text("First").tag(0)
                Text("Second").tag(1)
                Text("Third").tag(2)
            }
            .pickerStyle(.segmented)
            
            // Wheel style
            Picker("Wheel", selection: $selectedFruit) {
                ForEach(fruits, id: \.self) { fruit in
                    Text(fruit)
                }
            }
            .pickerStyle(.wheel)
        }
    }
}
```

### Toggle & Stepper

```swift
struct ControlExamples: View {
    @State private var isOn = false
    @State private var value = 0
    @State private var sliderValue = 0.5
    
    var body: some View {
        Form {
            // Toggle
            Toggle("Enable", isOn: $isOn)
            
            // Stepper
            Stepper("Value: \(value)", value: $value, in: 0...10)
            
            // Slider
            Slider(value: $sliderValue, in: 0...1)
            
            // Slider con label
            VStack {
                Text("Volume: \(Int(sliderValue * 100))%")
                Slider(value: $sliderValue)
            }
        }
    }
}
```

---

## Layout Containers

### Spacer

```swift
struct SpacerExample: View {
    var body: some View {
        VStack {
            Text("Top")
            Spacer() // Empuja contenido
            Text("Bottom")
        }
        
        HStack {
            Text("Left")
            Spacer()
            Text("Right")
        }
    }
}
```

### Divider

```swift
struct DividerExample: View {
    var body: some View {
        VStack {
            Text("Above")
            Divider()
            Text("Below")
        }
        
        HStack {
            Text("Left")
            Divider()
            Text("Right")
        }
    }
}
```

---

## Stacks

### VStack, HStack, ZStack

```swift
struct StackExamples: View {
    var body: some View {
        VStack {
            // VStack - Vertical
            VStack(alignment: .leading, spacing: 10) {
                Text("First")
                Text("Second")
                Text("Third")
            }
            
            Divider()
            
            // HStack - Horizontal
            HStack(alignment: .top, spacing: 20) {
                Image(systemName: "star")
                Text("Starred")
            }
            
            Divider()
            
            // ZStack - Overlay
            ZStack(alignment: .bottomTrailing) {
                Image(systemName: "photo")
                    .font(.system(size: 100))
                
                Image(systemName: "heart.fill")
                    .foregroundColor(.red)
                    .padding(8)
            }
        }
    }
}
```

### Grid (iOS 16+)

```swift
struct GridExample: View {
    let items = Array(1...10)
    
    var body: some View {
        Grid {
            GridRow {
                Text("Row 1 Col 1")
                Text("Row 1 Col 2")
            }
            
            GridRow {
                Text("Row 2 Col 1")
                    .gridCellColumns(2) // Span 2 columns
            }
        }
    }
}
```

---

## ScrollView y LazyStacks

### ScrollView

```swift
struct ScrollViewExample: View {
    var body: some View {
        ScrollView {
            VStack(spacing: 10) {
                ForEach(1...50, id: \.self) { number in
                    Text("Row \(number)")
                        .frame(maxWidth: .infinity)
                        .padding()
                        .background(Color.blue.opacity(0.1))
                        .cornerRadius(8)
                }
            }
            .padding()
        }
    }
}
```

### LazyVStack y LazyHStack

```swift
struct LazyStackExample: View {
    var body: some View {
        ScrollView {
            LazyVStack(spacing: 10) {
                ForEach(1...1000, id: \.self) { number in
                    Text("Row \(number)")
                        .frame(maxWidth: .infinity)
                        .padding()
                        .background(Color.green.opacity(0.1))
                        .onAppear {
                            print("Row \(number) appeared")
                        }
                }
            }
        }
    }
}
```

### LazyVGrid & LazyHGrid

```swift
struct GridExamples: View {
    let columns = [
        GridItem(.flexible()),
        GridItem(.flexible()),
        GridItem(.flexible())
    ]
    
    var body: some View {
        ScrollView {
            LazyVGrid(columns: columns, spacing: 10) {
                ForEach(1...100, id: \.self) { number in
                    RoundedRectangle(cornerRadius: 8)
                        .fill(Color.blue)
                        .frame(height: 80)
                        .overlay(Text("\(number)").foregroundColor(.white))
                }
            }
            .padding()
        }
    }
}
```

---

## Modifiers Comunes

### Frame & Padding

```swift
struct FramePaddingExample: View {
    var body: some View {
        VStack(spacing: 20) {
            // Frame fijo
            Text("Fixed")
                .frame(width: 200, height: 100)
                .background(Color.blue)
            
            // Frame máximo
            Text("Max Width")
                .frame(maxWidth: .infinity)
                .background(Color.green)
            
            // Padding
            Text("Padded")
                .padding()
                .background(Color.orange)
            
            // Padding específico
            Text("Custom Padding")
                .padding(.horizontal, 20)
                .padding(.vertical, 10)
                .background(Color.purple)
        }
    }
}
```

### Background & Overlay

```swift
struct BackgroundOverlayExample: View {
    var body: some View {
        VStack(spacing: 30) {
            // Background
            Text("Background")
                .padding()
                .background(
                    RoundedRectangle(cornerRadius: 10)
                        .fill(Color.blue)
                )
            
            // Overlay
            Circle()
                .fill(Color.gray)
                .frame(width: 100, height: 100)
                .overlay(
                    Image(systemName: "heart.fill")
                        .foregroundColor(.red)
                        .font(.largeTitle)
                )
            
            // Multiple layers
            Text("Layered")
                .padding()
                .background(Color.white)
                .cornerRadius(8)
                .overlay(
                    RoundedRectangle(cornerRadius: 8)
                        .stroke(Color.blue, lineWidth: 2)
                )
                .shadow(radius: 5)
        }
    }
}
```

### Shadow & Blur

```swift
struct EffectsExample: View {
    var body: some View {
        VStack(spacing: 30) {
            // Shadow
            Text("Shadow")
                .font(.title)
                .shadow(radius: 5)
            
            // Color shadow
            Text("Colored Shadow")
                .font(.title)
                .shadow(color: .red, radius: 10, x: 0, y: 5)
            
            // Blur
            Image(systemName: "star.fill")
                .font(.system(size: 50))
                .blur(radius: 2)
        }
    }
}
```

---

## Custom Modifiers

### ViewModifier

```swift
struct CardModifier: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding()
            .background(Color.white)
            .cornerRadius(10)
            .shadow(color: Color.black.opacity(0.1), radius: 5, x: 0, y: 2)
    }
}

extension View {
    func cardStyle() -> some View {
        modifier(CardModifier())
    }
}

// Uso
struct CardExample: View {
    var body: some View {
        VStack {
            Text("Card Content")
                .cardStyle()
            
            Image(systemName: "star")
                .font(.largeTitle)
                .cardStyle()
        }
    }
}
```

### Conditional Modifiers

```swift
extension View {
    @ViewBuilder
    func `if`<Transform: View>(_ condition: Bool, transform: (Self) -> Transform) -> some View {
        if condition {
            transform(self)
        } else {
            self
        }
    }
}

// Uso
struct ConditionalExample: View {
    @State private var isHighlighted = false
    
    var body: some View {
        Text("Conditional")
            .if(isHighlighted) { view in
                view.background(Color.yellow)
            }
    }
}
```

---

## Checklist de Aprendizaje

### Básico
- [ ] Text, Image, Button básicos
- [ ] TextField y SecureField
- [ ] VStack, HStack, ZStack
- [ ] Frame, padding, background

### Intermedio
- [ ] AsyncImage
- [ ] LazyStacks para performance
- [ ] Custom button styles
- [ ] Grid layouts
- [ ] Modifiers compuestos

### Avanzado
- [ ] Custom ViewModifiers
- [ ] Conditional modifiers
- [ ] Preference Keys
- [ ] GeometryReader
- [ ] ViewBuilder

---

## Próximos Pasos

### 1. Layout Avanzado
- GeometryReader
- Preference Keys
- Anchors
- Custom Layouts (iOS 16+)

### 2. Performance
- LazyStacks vs regular Stacks
- View identity
- Struct vs Class views

---

## Recursos

### Documentación
- 📚 [Views and Controls](https://developer.apple.com/documentation/swiftui/views-and-controls)
- 📖 [View Modifiers](https://developer.apple.com/documentation/swiftui/view-modifiers)

### Videos
- 🎥 [SwiftUI Views](https://www.youtube.com/watch?v=3krC2c56ceQ)
- 🎥 [Custom Modifiers](https://www.youtube.com/watch?v=rZVlQk8h7rk)

### Artículos
- 📝 [Hacking with Swift - Views](https://www.hackingwithswift.com/quick-start/swiftui/swiftui-tutorial-building-a-complete-project)
- 📝 [Swift by Sundell - ViewModifiers](https://www.swiftbysundell.com/articles/encapsulating-swiftui-view-styles/)

---

**Anterior:** [Navigation](Navigation.md)  
**Próximo:** [Lists & Forms](Lists-Forms.md)
