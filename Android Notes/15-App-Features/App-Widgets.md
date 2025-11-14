# App Widgets

## Widget Provider

```kotlin
class MyWidget : AppWidgetProvider() {
    override fun onUpdate(
        context: Context,
        appWidgetManager: AppWidgetManager,
        appWidgetIds: IntArray
    ) {
        appWidgetIds.forEach { widgetId ->
            val views = RemoteViews(context.packageName, R.layout.widget_layout)
            views.setTextViewText(R.id.widget_text, "Hello Widget")
            
            appWidgetManager.updateAppWidget(widgetId, views)
        }
    }
}
```

## Manifest

```xml
<receiver android:name=".MyWidget">
    <intent-filter>
        <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
    </intent-filter>
    <meta-data
        android:name="android.appwidget.provider"
        android:resource="@xml/widget_info" />
</receiver>
```

## Glance (Compose Widgets)

```kotlin
class MyGlanceWidget : GlanceAppWidget() {
    override suspend fun provideGlance(context: Context, id: GlanceId) {
        provideContent {
            Text("Hello from Glance!")
        }
    }
}
```

## Recursos
- [Widgets](https://developer.android.com/guide/topics/appwidgets/overview)
