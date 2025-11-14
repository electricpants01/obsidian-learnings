# URLSession en iOS - Guía Completa

## Tabla de Contenido
1. [¿Qué es URLSession?](#qué-es-urlsession)
2. [Configuración Básica](#configuración-básica)
3. [GET Requests](#get-requests)
4. [POST Requests](#post-requests)
5. [PUT y DELETE](#put-y-delete)
6. [Upload y Download](#upload-y-download)
7. [Error Handling](#error-handling)
8. [Authentication](#authentication)
9. [Async/Await](#asyncawait)
10. [Combine Integration](#combine-integration)
11. [API Client](#api-client)
12. [Best Practices](#best-practices)
13. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
14. [Próximos Pasos](#próximos-pasos)
15. [Recursos](#recursos)

---

## ¿Qué es URLSession?

**URLSession** es el API de Apple para realizar network requests. Proporciona una forma moderna y eficiente de descargar y subir datos.

### Conceptos Clave

```swift
// URLSession Components

// 1. URLSession
// - Coordina network tasks
// - Gestiona configuración

// 2. URLSessionConfiguration
// - Default, ephemeral, background
// - Headers, timeouts, caching

// 3. URLSessionTask
// - DataTask: Para data en memoria
// - DownloadTask: Para descargar archivos
// - UploadTask: Para subir archivos

// 4. URLRequest
// - Define la petición
// - URL, method, headers, body
```

---

## Configuración Básica

### URLSession Default

```swift
// Singleton compartido
let session = URLSession.shared

// Uso básico
let url = URL(string: "https://api.example.com/users")!

Task {
    let (data, response) = try await session.data(from: url)
    print("Received \(data.count) bytes")
}
```

### Custom Configuration

```swift
// Configuración personalizada
let configuration = URLSessionConfiguration.default
configuration.timeoutIntervalForRequest = 30
configuration.timeoutIntervalForResource = 300
configuration.allowsCellularAccess = true
configuration.waitsForConnectivity = true

// Headers globales
configuration.httpAdditionalHeaders = [
    "User-Agent": "MyApp/1.0",
    "Accept": "application/json"
]

// Crear sesión con configuración
let customSession = URLSession(configuration: configuration)
```

### Configuration Types

```swift
// 1. Default Configuration
let defaultConfig = URLSessionConfiguration.default
// - Usa caché en disco
// - Guarda cookies
// - Guarda credenciales

// 2. Ephemeral Configuration
let ephemeralConfig = URLSessionConfiguration.ephemeral
// - No guarda nada en disco
// - Ideal para navegación privada

// 3. Background Configuration
let backgroundConfig = URLSessionConfiguration.background(
    withIdentifier: "com.myapp.background"
)
// - Permite downloads en background
// - Continúa cuando app está suspendida
```

---

## GET Requests

### Simple GET

```swift
func fetchUsers() async throws -> [User] {
    let url = URL(string: "https://api.example.com/users")!
    
    let (data, _) = try await URLSession.shared.data(from: url)
    
    let decoder = JSONDecoder()
    let users = try decoder.decode([User].self, from: data)
    
    return users
}

struct User: Codable {
    let id: Int
    let name: String
    let email: String
}

// Uso
Task {
    let users = try await fetchUsers()
    print("Fetched \(users.count) users")
}
```

### GET con Query Parameters

```swift
func searchUsers(query: String, limit: Int = 20) async throws -> [User] {
    var components = URLComponents(string: "https://api.example.com/users/search")!
    
    components.queryItems = [
        URLQueryItem(name: "q", value: query),
        URLQueryItem(name: "limit", value: "\(limit)")
    ]
    
    guard let url = components.url else {
        throw NetworkError.invalidURL
    }
    
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode([User].self, from: data)
}

// Uso
Task {
    let results = try await searchUsers(query: "John", limit: 10)
    print("Found \(results.count) users")
}
```

### GET con Headers

```swift
func fetchProtectedData(token: String) async throws -> Data {
    let url = URL(string: "https://api.example.com/protected")!
    
    var request = URLRequest(url: url)
    request.httpMethod = "GET"
    request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
    request.setValue("application/json", forHTTPHeaderField: "Accept")
    
    let (data, response) = try await URLSession.shared.data(for: request)
    
    guard let httpResponse = response as? HTTPURLResponse else {
        throw NetworkError.invalidResponse
    }
    
    guard (200...299).contains(httpResponse.statusCode) else {
        throw NetworkError.httpError(httpResponse.statusCode)
    }
    
    return data
}

enum NetworkError: Error {
    case invalidURL
    case invalidResponse
    case httpError(Int)
}
```

---

## POST Requests

### POST JSON

```swift
struct CreateUserRequest: Codable {
    let name: String
    let email: String
    let password: String
}

struct CreateUserResponse: Codable {
    let id: Int
    let name: String
    let email: String
    let createdAt: String
}

func createUser(name: String, email: String, password: String) async throws -> CreateUserResponse {
    let url = URL(string: "https://api.example.com/users")!
    
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    
    let body = CreateUserRequest(name: name, email: email, password: password)
    request.httpBody = try JSONEncoder().encode(body)
    
    let (data, response) = try await URLSession.shared.data(for: request)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw NetworkError.invalidResponse
    }
    
    return try JSONDecoder().decode(CreateUserResponse.self, from: data)
}

// Uso
Task {
    let user = try await createUser(
        name: "John Doe",
        email: "john@example.com",
        password: "secure123"
    )
    print("Created user with ID: \(user.id)")
}
```

### POST Form Data

```swift
func login(email: String, password: String) async throws -> String {
    let url = URL(string: "https://api.example.com/login")!
    
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
    
    let bodyString = "email=\(email)&password=\(password)"
    request.httpBody = bodyString.data(using: .utf8)
    
    let (data, _) = try await URLSession.shared.data(for: request)
    
    struct LoginResponse: Codable {
        let token: String
    }
    
    let response = try JSONDecoder().decode(LoginResponse.self, from: data)
    return response.token
}
```

### POST Multipart Form Data

```swift
func uploadImage(_ image: UIImage, userId: String) async throws -> String {
    let url = URL(string: "https://api.example.com/upload")!
    
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    
    let boundary = UUID().uuidString
    request.setValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")
    
    var body = Data()
    
    // Add userId field
    body.append("--\(boundary)\r\n")
    body.append("Content-Disposition: form-data; name=\"userId\"\r\n\r\n")
    body.append("\(userId)\r\n")
    
    // Add image file
    if let imageData = image.jpegData(compressionQuality: 0.8) {
        body.append("--\(boundary)\r\n")
        body.append("Content-Disposition: form-data; name=\"image\"; filename=\"photo.jpg\"\r\n")
        body.append("Content-Type: image/jpeg\r\n\r\n")
        body.append(imageData)
        body.append("\r\n")
    }
    
    body.append("--\(boundary)--\r\n")
    
    request.httpBody = body
    
    let (data, _) = try await URLSession.shared.data(for: request)
    
    struct UploadResponse: Codable {
        let imageUrl: String
    }
    
    let response = try JSONDecoder().decode(UploadResponse.self, from: data)
    return response.imageUrl
}

extension Data {
    mutating func append(_ string: String) {
        if let data = string.data(using: .utf8) {
            append(data)
        }
    }
}
```

---

## PUT y DELETE

### PUT Request

```swift
struct UpdateUserRequest: Codable {
    let name: String
    let email: String
}

func updateUser(id: Int, name: String, email: String) async throws -> User {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    
    var request = URLRequest(url: url)
    request.httpMethod = "PUT"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    
    let body = UpdateUserRequest(name: name, email: email)
    request.httpBody = try JSONEncoder().encode(body)
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return try JSONDecoder().decode(User.self, from: data)
}
```

### DELETE Request

```swift
func deleteUser(id: Int) async throws {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    
    var request = URLRequest(url: url)
    request.httpMethod = "DELETE"
    
    let (_, response) = try await URLSession.shared.data(for: request)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw NetworkError.httpError(0)
    }
}

// Uso
Task {
    try await deleteUser(id: 123)
    print("User deleted successfully")
}
```

---

## Upload y Download

### Download File

```swift
func downloadFile(from urlString: String) async throws -> URL {
    let url = URL(string: urlString)!
    
    let (localURL, response) = try await URLSession.shared.download(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw NetworkError.invalidResponse
    }
    
    // Move to permanent location
    let documentsPath = FileManager.default.urls(
        for: .documentDirectory,
        in: .userDomainMask
    )[0]
    
    let destinationURL = documentsPath.appendingPathComponent(url.lastPathComponent)
    
    try? FileManager.default.removeItem(at: destinationURL)
    try FileManager.default.moveItem(at: localURL, to: destinationURL)
    
    return destinationURL
}

// Uso
Task {
    let fileURL = try await downloadFile(from: "https://example.com/file.pdf")
    print("Downloaded to: \(fileURL.path)")
}
```

### Download con Progress

```swift
class FileDownloader: NSObject, URLSessionDownloadDelegate {
    var progressHandler: ((Double) -> Void)?
    var completionHandler: ((Result<URL, Error>) -> Void)?
    
    private lazy var session: URLSession = {
        let config = URLSessionConfiguration.default
        return URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }()
    
    func download(from url: URL) {
        let task = session.downloadTask(with: url)
        task.resume()
    }
    
    // URLSessionDownloadDelegate
    func urlSession(
        _ session: URLSession,
        downloadTask: URLSessionDownloadTask,
        didFinishDownloadingTo location: URL
    ) {
        // Move file to permanent location
        let documentsPath = FileManager.default.urls(
            for: .documentDirectory,
            in: .userDomainMask
        )[0]
        
        let destinationURL = documentsPath.appendingPathComponent("downloaded_file")
        
        do {
            try? FileManager.default.removeItem(at: destinationURL)
            try FileManager.default.moveItem(at: location, to: destinationURL)
            completionHandler?(.success(destinationURL))
        } catch {
            completionHandler?(.failure(error))
        }
    }
    
    func urlSession(
        _ session: URLSession,
        downloadTask: URLSessionDownloadTask,
        didWriteData bytesWritten: Int64,
        totalBytesWritten: Int64,
        totalBytesExpectedToWrite: Int64
    ) {
        let progress = Double(totalBytesWritten) / Double(totalBytesExpectedToWrite)
        progressHandler?(progress)
    }
}

// Uso
let downloader = FileDownloader()
downloader.progressHandler = { progress in
    print("Progress: \(Int(progress * 100))%")
}
downloader.completionHandler = { result in
    switch result {
    case .success(let url):
        print("Downloaded to: \(url)")
    case .failure(let error):
        print("Error: \(error)")
    }
}

let url = URL(string: "https://example.com/largefile.zip")!
downloader.download(from: url)
```

### Upload File

```swift
func uploadFile(fileURL: URL) async throws -> String {
    let url = URL(string: "https://api.example.com/upload")!
    
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("application/octet-stream", forHTTPHeaderField: "Content-Type")
    
    let (data, response) = try await URLSession.shared.upload(for: request, fromFile: fileURL)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw NetworkError.invalidResponse
    }
    
    struct UploadResponse: Codable {
        let fileId: String
    }
    
    let uploadResponse = try JSONDecoder().decode(UploadResponse.self, from: data)
    return uploadResponse.fileId
}
```

---

## Error Handling

### Comprehensive Error Handling

```swift
enum APIError: Error {
    case invalidURL
    case requestFailed(Error)
    case invalidResponse
    case invalidData
    case decodingFailed(Error)
    case httpError(Int, String?)
    case unauthorized
    case notFound
    case serverError
    
    var localizedDescription: String {
        switch self {
        case .invalidURL:
            return "Invalid URL"
        case .requestFailed(let error):
            return "Request failed: \(error.localizedDescription)"
        case .invalidResponse:
            return "Invalid response from server"
        case .invalidData:
            return "Invalid data received"
        case .decodingFailed(let error):
            return "Failed to decode response: \(error.localizedDescription)"
        case .httpError(let code, let message):
            return "HTTP \(code): \(message ?? "Unknown error")"
        case .unauthorized:
            return "Unauthorized access"
        case .notFound:
            return "Resource not found"
        case .serverError:
            return "Server error occurred"
        }
    }
}

func fetchWithErrorHandling<T: Decodable>(url: URL) async throws -> T {
    do {
        let (data, response) = try await URLSession.shared.data(from: url)
        
        guard let httpResponse = response as? HTTPURLResponse else {
            throw APIError.invalidResponse
        }
        
        switch httpResponse.statusCode {
        case 200...299:
            do {
                return try JSONDecoder().decode(T.self, from: data)
            } catch {
                throw APIError.decodingFailed(error)
            }
        case 401:
            throw APIError.unauthorized
        case 404:
            throw APIError.notFound
        case 500...599:
            throw APIError.serverError
        default:
            let message = String(data: data, encoding: .utf8)
            throw APIError.httpError(httpResponse.statusCode, message)
        }
    } catch let error as APIError {
        throw error
    } catch {
        throw APIError.requestFailed(error)
    }
}
```

---

## Authentication

### Bearer Token

```swift
class AuthenticatedAPIClient {
    private var accessToken: String?
    private let session: URLSession
    
    init() {
        let config = URLSessionConfiguration.default
        self.session = URLSession(configuration: config)
    }
    
    func setAccessToken(_ token: String) {
        accessToken = token
    }
    
    func request<T: Decodable>(
        _ endpoint: String,
        method: String = "GET",
        body: Encodable? = nil
    ) async throws -> T {
        guard let url = URL(string: "https://api.example.com/\(endpoint)") else {
            throw APIError.invalidURL
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = method
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        if let token = accessToken {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }
        
        if let body = body {
            request.httpBody = try JSONEncoder().encode(body)
        }
        
        let (data, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            throw APIError.invalidResponse
        }
        
        return try JSONDecoder().decode(T.self, from: data)
    }
}

// Uso
let client = AuthenticatedAPIClient()
client.setAccessToken("your_access_token")

Task {
    let users: [User] = try await client.request("users")
    print("Fetched \(users.count) users")
}
```

### Basic Authentication

```swift
func fetchWithBasicAuth(username: String, password: String) async throws -> Data {
    let url = URL(string: "https://api.example.com/data")!
    
    var request = URLRequest(url: url)
    
    let credentials = "\(username):\(password)"
    let credentialsData = credentials.data(using: .utf8)!
    let base64Credentials = credentialsData.base64EncodedString()
    
    request.setValue("Basic \(base64Credentials)", forHTTPHeaderField: "Authorization")
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return data
}
```

---

## Async/Await

### Modern Async Approach

```swift
struct APIClient {
    func fetchUsers() async throws -> [User] {
        let url = URL(string: "https://api.example.com/users")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode([User].self, from: data)
    }
    
    func fetchUser(id: Int) async throws -> User {
        let url = URL(string: "https://api.example.com/users/\(id)")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode(User.self, from: data)
    }
    
    func fetchMultipleUsers(ids: [Int]) async throws -> [User] {
        try await withThrowingTaskGroup(of: User.self) { group in
            for id in ids {
                group.addTask {
                    try await self.fetchUser(id: id)
                }
            }
            
            var users: [User] = []
            for try await user in group {
                users.append(user)
            }
            return users
        }
    }
}

// Uso
let client = APIClient()

Task {
    // Sequential
    let users = try await client.fetchUsers()
    
    // Parallel
    async let user1 = client.fetchUser(id: 1)
    async let user2 = client.fetchUser(id: 2)
    async let user3 = client.fetchUser(id: 3)
    
    let results = try await [user1, user2, user3]
    print("Fetched \(results.count) users in parallel")
}
```

---

## Combine Integration

### URLSession + Combine

```swift
import Combine

extension URLSession {
    func fetchPublisher<T: Decodable>(
        _ url: URL,
        type: T.Type
    ) -> AnyPublisher<T, Error> {
        return dataTaskPublisher(for: url)
            .map(\.data)
            .decode(type: T.self, decoder: JSONDecoder())
            .eraseToAnyPublisher()
    }
}

// Uso
class ViewModel: ObservableObject {
    @Published var users: [User] = []
    private var cancellables = Set<AnyCancellable>()
    
    func loadUsers() {
        let url = URL(string: "https://api.example.com/users")!
        
        URLSession.shared.fetchPublisher(url, type: [User].self)
            .receive(on: DispatchQueue.main)
            .sink(
                receiveCompletion: { completion in
                    if case .failure(let error) = completion {
                        print("Error: \(error)")
                    }
                },
                receiveValue: { [weak self] users in
                    self?.users = users
                }
            )
            .store(in: &cancellables)
    }
}
```

---

## API Client

### Complete API Client

```swift
protocol APIClientProtocol {
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T
}

enum Endpoint {
    case users
    case user(id: Int)
    case createUser(name: String, email: String)
    case updateUser(id: Int, name: String)
    case deleteUser(id: Int)
    
    var url: URL {
        let base = "https://api.example.com"
        switch self {
        case .users:
            return URL(string: "\(base)/users")!
        case .user(let id):
            return URL(string: "\(base)/users/\(id)")!
        case .createUser, .updateUser, .deleteUser:
            return URL(string: "\(base)/users")!
        }
    }
    
    var method: String {
        switch self {
        case .users, .user:
            return "GET"
        case .createUser:
            return "POST"
        case .updateUser:
            return "PUT"
        case .deleteUser:
            return "DELETE"
        }
    }
    
    var body: Encodable? {
        switch self {
        case .createUser(let name, let email):
            return ["name": name, "email": email]
        case .updateUser(_, let name):
            return ["name": name]
        default:
            return nil
        }
    }
}

class APIClient: APIClientProtocol {
    private let session: URLSession
    private var accessToken: String?
    
    init(session: URLSession = .shared) {
        self.session = session
    }
    
    func setAccessToken(_ token: String) {
        accessToken = token
    }
    
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        var request = URLRequest(url: endpoint.url)
        request.httpMethod = endpoint.method
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        if let token = accessToken {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }
        
        if let body = endpoint.body {
            request.httpBody = try JSONEncoder().encode(body)
        }
        
        let (data, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse else {
            throw APIError.invalidResponse
        }
        
        guard (200...299).contains(httpResponse.statusCode) else {
            throw APIError.httpError(httpResponse.statusCode, nil)
        }
        
        return try JSONDecoder().decode(T.self, from: data)
    }
}

// Uso
let client = APIClient()

Task {
    let users: [User] = try await client.request(.users)
    let user: User = try await client.request(.user(id: 1))
}
```

---

## Best Practices

### 1. Use async/await

```swift
// ✅ Bueno - async/await
func fetchData() async throws -> Data {
    let url = URL(string: "https://api.example.com/data")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return data
}
```

### 2. Reuse URLSession

```swift
// ✅ Bueno - Reutilizar sesión
class NetworkManager {
    static let shared = NetworkManager()
    private let session: URLSession
    
    private init() {
        let config = URLSessionConfiguration.default
        self.session = URLSession(configuration: config)
    }
}
```

### 3. Handle Errors Properly

```swift
// ✅ Bueno - Error handling específico
do {
    let data = try await fetchData()
} catch let error as APIError {
    // Handle specific API errors
} catch {
    // Handle general errors
}
```

---

## Checklist de Aprendizaje

### Básico
- [ ] GET requests simples
- [ ] POST con JSON
- [ ] Error handling básico
- [ ] Decodificar respuestas

### Intermedio
- [ ] Query parameters
- [ ] Headers personalizados
- [ ] Upload/Download files
- [ ] Authentication

### Avanzado
- [ ] Custom API Client
- [ ] Retry logic
- [ ] Request interceptors
- [ ] Performance optimization

---

## Próximos Pasos

### 1. Advanced Features
- Caching strategies
- Request prioritization
- Network monitoring

### 2. Third-Party Libraries
- Alamofire
- Moya
- Apollo (GraphQL)

---

## Recursos

### Documentación
- 📚 [URLSession](https://developer.apple.com/documentation/foundation/urlsession)
- 📖 [Networking Overview](https://developer.apple.com/documentation/foundation/url_loading_system)

### Videos
- 🎥 [WWDC: Networking](https://developer.apple.com/videos/play/wwdc2021/10095/)

### Artículos
- 📝 [URLSession Tutorial](https://www.raywenderlich.com/3244963-urlsession-tutorial-getting-started)
- 📝 [Modern Networking](https://www.swiftbysundell.com/articles/networking-in-swift/)

---

**Anterior:** [Actors](../05-Concurrency/Actors.md)  
**Próximo:** [CoreData](../07-Data-Persistence/CoreData.md)
