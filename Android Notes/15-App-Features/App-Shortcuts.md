# App Shortcuts

## Static Shortcuts

```xml
<!-- res/xml/shortcuts.xml -->
<shortcuts xmlns:android="http://schemas.android.com/apk/res/android">
    <shortcut
        android:shortcutId="compose"
        android:enabled="true"
        android:icon="@drawable/ic_compose"
        android:shortcutShortLabel="@string/compose_short"
        android:shortcutLongLabel="@string/compose_long">
        <intent
            android:action="android.intent.action.VIEW"
            android:targetPackage="com.example.app"
            android:targetClass="com.example.app.ComposeActivity" />
    </shortcut>
</shortcuts>

<!-- AndroidManifest.xml -->
<activity android:name=".MainActivity">
    <meta-data
        android:name="android.app.shortcuts"
        android:resource="@xml/shortcuts" />
</activity>
```

## Dynamic Shortcuts

```kotlin
val shortcut = ShortcutInfo.Builder(context, "id1")
    .setShortLabel("Web site")
    .setLongLabel("Open the web site")
    .setIcon(Icon.createWithResource(context, R.drawable.icon))
    .setIntent(Intent(Intent.ACTION_VIEW, Uri.parse("https://www.example.com/")))
    .build()

ShortcutManagerCompat.pushDynamicShortcut(context, shortcut)
```

## Recursos
- [Shortcuts](https://developer.android.com/guide/topics/ui/shortcuts)
