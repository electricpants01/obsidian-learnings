# Best Practices

## Arquitectura

- Usa Clean Architecture con capas separadas (UI, Domain, Data)
- Implementa MVVM o MVI para separar lógica de UI
- Single Source of Truth para manejo de datos
- Repository Pattern para abstracción de datos

## Código

- Sigue principios SOLID
- DRY (Don't Repeat Yourself)
- KISS (Keep It Simple, Stupid)
- Naming conventions claras y descriptivas
- Code reviews regulares

## Performance

- Evita memory leaks
- Usa herramientas de profiling
- Optimiza recomposiciones en Compose
- Lazy loading cuando sea posible
- Baseline Profiles para apps en producción

## Testing

- Unit tests para lógica de negocio
- Integration tests para flujos completos
- UI tests para interacciones críticas
- Mínimo 70% code coverage

## Security

- Nunca almacenar credenciales en código
- Usar EncryptedSharedPreferences
- Implementar SSL Pinning
- Validar input del usuario
- Ofuscar código con R8/ProGuard

## UI/UX

- Seguir Material Design Guidelines
- Soporte para dark mode
- Accesibilidad (TalkBack, content descriptions)
- Internacionalización (i18n)
- Responsive design para tablets y foldables

## Recursos
- [Best Practices](https://developer.android.com/topic/architecture/recommendations)
