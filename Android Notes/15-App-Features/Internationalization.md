# Internationalization (i18n)

## String Resources

```xml
<!-- res/values/strings.xml -->
<resources>
    <string name="app_name">My App</string>
    <string name="welcome">Welcome</string>
</resources>

<!-- res/values-es/strings.xml -->
<resources>
    <string name="app_name">Mi App</string>
    <string name="welcome">Bienvenido</string>
</resources>
```

## Uso

```kotlin
// En código
val welcome = context.getString(R.string.welcome)

// En Compose
@Composable
fun WelcomeText() {
    Text(stringResource(R.string.welcome))
}
```

## Plurals

```xml
<plurals name="numberOfItems">
    <item quantity="one">%d item</item>
    <item quantity="other">%d items</item>
</plurals>
```

## Recursos
- [i18n](https://developer.android.com/guide/topics/resources/localization)
