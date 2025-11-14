# iOS Best Practices

## Arquitectura
- ✅ Usa MVVM para apps SwiftUI
- ✅ Separa lógica de negocio de UI
- ✅ Implementa Repository Pattern

## Código
- ✅ Prefiere `let` sobre `var`
- ✅ Usa guard para early returns
- ✅ Evita force unwrapping (!)
- ✅ Implementa protocolos para testing

## SwiftUI
- ✅ Usa @State para estado local
- ✅ Usa @StateObject para ViewModels
- ✅ Mantén vistas pequeñas y reutilizables
- ✅ Usa @MainActor para ViewModels

## Concurrencia
- ✅ Usa async/await sobre callbacks
- ✅ Marca ViewModels con @MainActor
- ✅ Usa Actors para estado compartido

## Performance
- ✅ Lazy loading de imágenes
- ✅ Usa Instruments regularmente
- ✅ Evita retain cycles

## Seguridad
- ✅ Usa Keychain para datos sensibles
- ✅ Implementa biometric auth
- ✅ Valida inputs

## Testing
- ✅ Unit tests para lógica de negocio
- ✅ UI tests para flows críticos
- ✅ Usa mocks para networking

## Recursos
- [Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)
- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines)
