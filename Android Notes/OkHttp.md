# OkHttp

## Setup

```kotlin
dependencies {
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
    implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")
}
```

## Interceptors

```kotlin
val loggingInterceptor = HttpLoggingInterceptor().apply {
    level = HttpLoggingInterceptor.Level.BODY
}

val authInterceptor = Interceptor { chain ->
    val request = chain.request().newBuilder()
        .addHeader("Authorization", "Bearer $token")
        .build()
    chain.proceed(request)
}

val client = OkHttpClient.Builder()
    .addInterceptor(loggingInterceptor)
    .addInterceptor(authInterceptor)
    .connectTimeout(30, TimeUnit.SECONDS)
    .readTimeout(30, TimeUnit.SECONDS)
    .build()

val retrofit = Retrofit.Builder()
    .client(client)
    .baseUrl("https://api.example.com/")
    .build()
```

## Recursos
- [OkHttp](https://square.github.io/okhttp/)
