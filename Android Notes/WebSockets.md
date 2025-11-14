# WebSockets

## OkHttp WebSocket

```kotlin
val client = OkHttpClient()

val request = Request.Builder()
    .url("wss://echo.websocket.org")
    .build()

val listener = object : WebSocketListener() {
    override fun onOpen(webSocket: WebSocket, response: Response) {
        webSocket.send("Hello!")
    }
    
    override fun onMessage(webSocket: WebSocket, text: String) {
        println("Received: $text")
    }
    
    override fun onFailure(webSocket: WebSocket, t: Throwable, response: Response?) {
        println("Error: ${t.message}")
    }
}

val webSocket = client.newWebSocket(request, listener)
```

## Ktor WebSocket

```kotlin
val client = HttpClient {
    install(WebSockets)
}

client.webSocket("wss://echo.websocket.org") {
    send("Hello!")
    for (frame in incoming) {
        when (frame) {
            is Frame.Text -> println(frame.readText())
        }
    }
}
```

## Recursos
- [WebSockets](https://developer.android.com/guide/topics/connectivity/websockets)
