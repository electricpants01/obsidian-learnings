# URLSession - Networking en iOS

## Tabla de Contenido
1. [GET Request](#get-request)
2. [POST Request](#post-request)
3. [Error Handling](#error-handling)
4. [Async/Await](#asyncawait)
5. [Download/Upload](#downloadupload)
6. [Checklist](#checklist-de-aprendizaje)
7. [Recursos](#recursos)

---

## GET Request

```swift
// Async/Await
func fetchUsers() async throws -> [User] {
    let url = URL(string: "https://api.example.com/users")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode([User].self, from: data)
}

// Con headers
func fetchWithAuth() async throws -> Data {
    var request = URLRequest(url: URL(string: "https://api.example.com/data")!)
    request.setValue("Bearer token", forHTTPHeaderField: "Authorization")
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return data
}
```

---

## POST Request

```swift
struct CreateUserRequest: Codable {
    let name: String
    let email: String
}

func createUser(name: String, email: String) async throws -> User {
    let url = URL(string: "https://api.example.com/users")!
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    
    let body = CreateUserRequest(name: name, email: email)
    request.httpBody = try JSONEncoder().encode(body)
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return try JSONDecoder().decode(User.self, from: data)
}
```

---

## Error Handling

```swift
enum NetworkError: Error {
    case invalidURL
    case invalidResponse
    case unauthorized
    case serverError(Int)
    case decodingError
}

func fetchData<T: Decodable>(_ type: T.Type, from url: URL) async throws -> T {
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse else {
        throw NetworkError.invalidResponse
    }
    
    switch httpResponse.statusCode {
    case 200...299:
        do {
            return try JSONDecoder().decode(T.self, from: data)
        } catch {
            throw NetworkError.decodingError
        }
    case 401:
        throw NetworkError.unauthorized
    default:
        throw NetworkError.serverError(httpResponse.statusCode)
    }
}
```

---

## Async/Await

```swift
class APIClient {
    static let shared = APIClient()
    private init() {}
    
    func request<T: Decodable>(
        _ endpoint: String,
        method: String = "GET",
        body: Encodable? = nil
    ) async throws -> T {
        guard let url = URL(string: "https://api.example.com/\(endpoint)") else {
            throw NetworkError.invalidURL
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = method
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        if let body = body {
            request.httpBody = try JSONEncoder().encode(body)
        }
        
        let (data, response) = try await URLSession.shared.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.invalidResponse
        }
        
        return try JSONDecoder().decode(T.self, from: data)
    }
}

// Uso
let users: [User] = try await APIClient.shared.request("users")
```

---

## Download/Upload

```swift
// Download
func downloadFile(from url: URL) async throws -> URL {
    let (localURL, _) = try await URLSession.shared.download(from: url)
    return localURL
}

// Upload
func uploadImage(_ image: UIImage) async throws -> Response {
    let url = URL(string: "https://api.example.com/upload")!
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    
    let imageData = image.jpegData(compressionQuality: 0.8)!
    let (data, _) = try await URLSession.shared.upload(for: request, from: imageData)
    
    return try JSONDecoder().decode(Response.self, from: data)
}
```

---

## Checklist de Aprendizaje

### Básico
- [ ] GET requests con URLSession
- [ ] Decodificar JSON con Codable
- [ ] Manejar errores básicos

### Intermedio
- [ ] POST/PUT/DELETE requests
- [ ] Headers y autenticación
- [ ] Error handling robusto
- [ ] Download/Upload

### Avanzado
- [ ] Custom URLSession configuration
- [ ] Certificate pinning
- [ ] Retry logic
- [ ] Request/Response interceptors

---

## Recursos

- 📚 [URLSession](https://developer.apple.com/documentation/foundation/urlsession)
- 🎥 [Networking in Swift](https://www.youtube.com/watch?v=s3b-5O5JUec)
- 📝 [Hacking with Swift - URLSession](https://www.hackingwithswift.com/articles/153/how-to-test-ios-networking-code-the-easy-way)
