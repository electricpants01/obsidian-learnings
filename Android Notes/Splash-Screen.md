# Splash Screen

## Setup (Android 12+)

```xml
<!-- res/values/themes.xml -->
<style name="Theme.App" parent="Theme.SplashScreen">
    <item name="windowSplashScreenBackground">@color/splash_bg</item>
    <item name="windowSplashScreenAnimatedIcon">@drawable/ic_launcher</item>
    <item name="postSplashScreenTheme">@style/Theme.App.Main</item>
</style>
```

## Activity

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        val splashScreen = installSplashScreen()
        super.onCreate(savedInstanceState)
        
        splashScreen.setKeepOnScreenCondition { false }
    }
}
```

## Recursos
- [Splash Screen](https://developer.android.com/guide/topics/ui/splash-screen)
