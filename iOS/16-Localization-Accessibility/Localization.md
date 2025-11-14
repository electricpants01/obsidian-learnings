# Localization

## Setup

```swift
// Localizable.strings (en)
"welcome" = "Welcome";

// Localizable.strings (es)
"welcome" = "Bienvenido";
```

## Uso

```swift
Text(NSLocalizedString("welcome", comment: ""))
// o
Text("welcome")
```

## Recursos
- [Localization](https://developer.apple.com/documentation/xcode/localization)
