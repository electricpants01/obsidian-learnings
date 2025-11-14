# WebView

## Básico

```kotlin
@Composable
fun WebViewScreen(url: String) {
    AndroidView(factory = { context ->
        WebView(context).apply {
            settings.javaScriptEnabled = true
            webViewClient = WebViewClient()
            loadUrl(url)
        }
    })
}
```

## Con Estado

```kotlin
val webView = rememberWebViewState(url = "https://example.com")

WebView(
    state = webView,
    onCreated = { webView ->
        webView.settings.javaScriptEnabled = true
    }
)
```

## Recursos
- [WebView](https://developer.android.com/reference/android/webkit/WebView)
