# CoreData en iOS - Guía Completa

## Tabla de Contenido
1. [¿Qué es CoreData?](#qué-es-coredata)
2. [Setup Inicial](#setup-inicial)
3. [Data Model](#data-model)
4. [NSManagedObject](#nsmanagedobject)
5. [CRUD Operations](#crud-operations)
6. [Fetch Requests](#fetch-requests)
7. [Predicates](#predicates)
8. [Relationships](#relationships)
9. [Background Context](#background-context)
10. [Migration](#migration)
11. [Best Practices](#best-practices)
12. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
13. [Próximos Pasos](#próximos-pasos)
14. [Recursos](#recursos)

---

## ¿Qué es CoreData?

**CoreData** es el framework de Apple para gestionar el modelo de objetos y persistencia de datos en iOS/macOS.

### Arquitectura

```
┌─────────────────────────────────────┐
│      NSManagedObjectContext         │
│   (Working memory para objetos)     │
└───────────┬─────────────────────────┘
            │
┌───────────▼─────────────────────────┐
│  NSPersistentStoreCoordinator       │
│   (Coordina stores)                 │
└───────────┬─────────────────────────┘
            │
┌───────────▼─────────────────────────┐
│    NSPersistentStore                │
│   (SQLite, Binary, In-Memory)       │
└─────────────────────────────────────┘
```

---

## Setup Inicial

### CoreData Stack Manual

```swift
import CoreData

class CoreDataStack {
    static let shared = CoreDataStack()
    
    private init() {}
    
    // Persistent Container
    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "MyApp")
        
        container.loadPersistentStores { description, error in
            if let error = error {
                fatalError("Unable to load persistent stores: \(error)")
            }
        }
        
        return container
    }()
    
    // Main Context
    var context: NSManagedObjectContext {
        return persistentContainer.viewContext
    }
    
    // Save Context
    func saveContext() {
        let context = persistentContainer.viewContext
        
        if context.hasChanges {
            do {
                try context.save()
            } catch {
                print("Error saving context: \(error)")
            }
        }
    }
}
```

### CoreData Stack con App Delegate

```swift
@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func applicationWillTerminate(_ application: UIApplication) {
        CoreDataStack.shared.saveContext()
    }
}
```

### Setup con SwiftUI

```swift
@main
struct MyApp: App {
    let persistenceController = PersistenceController.shared
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(\.managedObjectContext, persistenceController.container.viewContext)
        }
    }
}

struct PersistenceController {
    static let shared = PersistenceController()
    
    let container: NSPersistentContainer
    
    init(inMemory: Bool = false) {
        container = NSPersistentContainer(name: "MyApp")
        
        if inMemory {
            container.persistentStoreDescriptions.first!.url = URL(fileURLWithPath: "/dev/null")
        }
        
        container.loadPersistentStores { storeDescription, error in
            if let error = error as NSError? {
                fatalError("Unresolved error \(error), \(error.userInfo)")
            }
        }
        
        container.viewContext.automaticallyMergesChangesFromParent = true
    }
}
```

---

## Data Model

### Creating Entities

```swift
// En el .xcdatamodeld, crear Entity "User" con attributes:
// - id: UUID
// - name: String
// - email: String
// - createdAt: Date

// Generar NSManagedObject subclass
// Editor -> Create NSManagedObject Subclass...
```

### Generated NSManagedObject

```swift
import CoreData

@objc(User)
public class User: NSManagedObject {
    @NSManaged public var id: UUID
    @NSManaged public var name: String
    @NSManaged public var email: String
    @NSManaged public var createdAt: Date
}

extension User: Identifiable {
    @nonobjc public class func fetchRequest() -> NSFetchRequest<User> {
        return NSFetchRequest<User>(entityName: "User")
    }
}
```

### Custom NSManagedObject

```swift
@objc(User)
public class User: NSManagedObject {
    @NSManaged public var id: UUID
    @NSManaged public var name: String
    @NSManaged public var email: String
    @NSManaged public var createdAt: Date
    
    // Computed properties
    var initials: String {
        let components = name.components(separatedBy: " ")
        return components.compactMap { $0.first }.map { String($0) }.joined()
    }
    
    var formattedDate: String {
        let formatter = DateFormatter()
        formatter.dateStyle = .medium
        return formatter.string(from: createdAt)
    }
    
    // Convenience initializer
    convenience init(context: NSManagedObjectContext, name: String, email: String) {
        self.init(context: context)
        self.id = UUID()
        self.name = name
        self.email = email
        self.createdAt = Date()
    }
}
```

---

## NSManagedObject

### Insert Object

```swift
func createUser(name: String, email: String) {
    let context = CoreDataStack.shared.context
    
    let user = User(context: context)
    user.id = UUID()
    user.name = name
    user.email = email
    user.createdAt = Date()
    
    CoreDataStack.shared.saveContext()
}

// O con convenience initializer
func createUserConvenience(name: String, email: String) {
    let context = CoreDataStack.shared.context
    let user = User(context: context, name: name, email: email)
    CoreDataStack.shared.saveContext()
}
```

### Update Object

```swift
func updateUser(_ user: User, name: String, email: String) {
    user.name = name
    user.email = email
    
    CoreDataStack.shared.saveContext()
}
```

### Delete Object

```swift
func deleteUser(_ user: User) {
    let context = CoreDataStack.shared.context
    context.delete(user)
    
    CoreDataStack.shared.saveContext()
}
```

---

## CRUD Operations

### Complete CRUD Manager

```swift
class UserManager {
    private let context = CoreDataStack.shared.context
    
    // CREATE
    func create(name: String, email: String) -> User {
        let user = User(context: context, name: name, email: email)
        saveContext()
        return user
    }
    
    // READ
    func fetchAll() -> [User] {
        let request = User.fetchRequest()
        
        do {
            return try context.fetch(request)
        } catch {
            print("Error fetching users: \(error)")
            return []
        }
    }
    
    func fetch(by id: UUID) -> User? {
        let request = User.fetchRequest()
        request.predicate = NSPredicate(format: "id == %@", id as CVarArg)
        request.fetchLimit = 1
        
        do {
            return try context.fetch(request).first
        } catch {
            print("Error fetching user: \(error)")
            return nil
        }
    }
    
    // UPDATE
    func update(_ user: User, name: String, email: String) {
        user.name = name
        user.email = email
        saveContext()
    }
    
    // DELETE
    func delete(_ user: User) {
        context.delete(user)
        saveContext()
    }
    
    func deleteAll() {
        let request: NSFetchRequest<NSFetchRequestResult> = User.fetchRequest()
        let deleteRequest = NSBatchDeleteRequest(fetchRequest: request)
        
        do {
            try context.execute(deleteRequest)
            saveContext()
        } catch {
            print("Error deleting all users: \(error)")
        }
    }
    
    // SAVE
    private func saveContext() {
        CoreDataStack.shared.saveContext()
    }
}

// Uso
let manager = UserManager()

// Create
let user = manager.create(name: "John Doe", email: "john@example.com")

// Read
let users = manager.fetchAll()
if let foundUser = manager.fetch(by: user.id) {
    print("Found: \(foundUser.name)")
}

// Update
manager.update(user, name: "Jane Doe", email: "jane@example.com")

// Delete
manager.delete(user)
```

---

## Fetch Requests

### Basic Fetch Request

```swift
func fetchUsers() -> [User] {
    let context = CoreDataStack.shared.context
    let request = User.fetchRequest()
    
    do {
        return try context.fetch(request)
    } catch {
        print("Error: \(error)")
        return []
    }
}
```

### Fetch con Sort Descriptors

```swift
func fetchUsersSorted() -> [User] {
    let context = CoreDataStack.shared.context
    let request = User.fetchRequest()
    
    // Sort by name ascending
    request.sortDescriptors = [
        NSSortDescriptor(key: "name", ascending: true)
    ]
    
    // Multiple sort descriptors
    request.sortDescriptors = [
        NSSortDescriptor(key: "createdAt", ascending: false),
        NSSortDescriptor(key: "name", ascending: true)
    ]
    
    do {
        return try context.fetch(request)
    } catch {
        print("Error: \(error)")
        return []
    }
}
```

### Fetch con Limit

```swift
func fetchRecentUsers(limit: Int = 10) -> [User] {
    let context = CoreDataStack.shared.context
    let request = User.fetchRequest()
    
    request.sortDescriptors = [NSSortDescriptor(key: "createdAt", ascending: false)]
    request.fetchLimit = limit
    
    do {
        return try context.fetch(request)
    } catch {
        print("Error: \(error)")
        return []
    }
}
```

### NSFetchedResultsController (UIKit)

```swift
class UsersViewController: UITableViewController {
    var fetchedResultsController: NSFetchedResultsController<User>!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupFetchedResultsController()
    }
    
    func setupFetchedResultsController() {
        let context = CoreDataStack.shared.context
        let request = User.fetchRequest()
        request.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
        
        fetchedResultsController = NSFetchedResultsController(
            fetchRequest: request,
            managedObjectContext: context,
            sectionNameKeyPath: nil,
            cacheName: nil
        )
        
        fetchedResultsController.delegate = self
        
        do {
            try fetchedResultsController.performFetch()
        } catch {
            print("Error: \(error)")
        }
    }
    
    // MARK: - Table View
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return fetchedResultsController.sections?[section].numberOfObjects ?? 0
    }
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
        let user = fetchedResultsController.object(at: indexPath)
        cell.textLabel?.text = user.name
        return cell
    }
}

// MARK: - NSFetchedResultsControllerDelegate
extension UsersViewController: NSFetchedResultsControllerDelegate {
    func controllerDidChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
        tableView.reloadData()
    }
}
```

---

## Predicates

### Simple Predicates

```swift
// Exact match
let predicate = NSPredicate(format: "name == %@", "John")

// Contains
let predicate = NSPredicate(format: "name CONTAINS[cd] %@", "john")
// [c] = case insensitive
// [d] = diacritic insensitive

// Begins with
let predicate = NSPredicate(format: "email BEGINSWITH %@", "john")

// Ends with
let predicate = NSPredicate(format: "email ENDSWITH %@", "@example.com")

// Multiple conditions (AND)
let predicate = NSPredicate(
    format: "name CONTAINS[cd] %@ AND email CONTAINS %@",
    "john", "@example.com"
)

// Multiple conditions (OR)
let predicate = NSPredicate(
    format: "name == %@ OR email == %@",
    "John", "john@example.com"
)
```

### Complex Predicates

```swift
func searchUsers(query: String) -> [User] {
    let context = CoreDataStack.shared.context
    let request = User.fetchRequest()
    
    let namePredicate = NSPredicate(format: "name CONTAINS[cd] %@", query)
    let emailPredicate = NSPredicate(format: "email CONTAINS[cd] %@", query)
    
    request.predicate = NSCompoundPredicate(
        orPredicateWithSubpredicates: [namePredicate, emailPredicate]
    )
    
    do {
        return try context.fetch(request)
    } catch {
        print("Error: \(error)")
        return []
    }
}

// Date predicates
func fetchRecentUsers(since date: Date) -> [User] {
    let context = CoreDataStack.shared.context
    let request = User.fetchRequest()
    
    request.predicate = NSPredicate(format: "createdAt >= %@", date as NSDate)
    
    do {
        return try context.fetch(request)
    } catch {
        print("Error: \(error)")
        return []
    }
}
```

---

## Relationships

### One-to-Many Relationship

```swift
// Entities:
// User: name, email
// Post: title, content, createdAt
// Relationship: User.posts <- inverse -> Post.author

@objc(User)
public class User: NSManagedObject {
    @NSManaged public var name: String
    @NSManaged public var email: String
    @NSManaged public var posts: NSSet?
    
    var postsArray: [Post] {
        let set = posts as? Set<Post> ?? []
        return set.sorted { $0.createdAt > $1.createdAt }
    }
}

@objc(Post)
public class Post: NSManagedObject {
    @NSManaged public var title: String
    @NSManaged public var content: String
    @NSManaged public var createdAt: Date
    @NSManaged public var author: User?
}

// Create relationship
func createPost(for user: User, title: String, content: String) {
    let context = CoreDataStack.shared.context
    
    let post = Post(context: context)
    post.title = title
    post.content = content
    post.createdAt = Date()
    post.author = user
    
    CoreDataStack.shared.saveContext()
}

// Fetch with relationship
func fetchUserPosts(_ user: User) -> [Post] {
    let context = CoreDataStack.shared.context
    let request = Post.fetchRequest()
    request.predicate = NSPredicate(format: "author == %@", user)
    request.sortDescriptors = [NSSortDescriptor(key: "createdAt", ascending: false)]
    
    do {
        return try context.fetch(request)
    } catch {
        return []
    }
}
```

### Many-to-Many Relationship

```swift
// Entities:
// Student: name
// Course: title
// Relationship: Student.courses <-> Course.students

@objc(Student)
public class Student: NSManagedObject {
    @NSManaged public var name: String
    @NSManaged public var courses: NSSet?
    
    func addToCourses(_ course: Course) {
        let courses = self.mutableSetValue(forKey: "courses")
        courses.add(course)
    }
    
    func removeFromCourses(_ course: Course) {
        let courses = self.mutableSetValue(forKey: "courses")
        courses.remove(course)
    }
}

@objc(Course)
public class Course: NSManagedObject {
    @NSManaged public var title: String
    @NSManaged public var students: NSSet?
}

// Usage
func enrollStudent(_ student: Student, in course: Course) {
    student.addToCourses(course)
    CoreDataStack.shared.saveContext()
}
```

---

## Background Context

### Background Operations

```swift
func performBackgroundTask() {
    let context = CoreDataStack.shared.persistentContainer.newBackgroundContext()
    
    context.perform {
        // Create objects in background
        for i in 0..<1000 {
            let user = User(context: context)
            user.id = UUID()
            user.name = "User \(i)"
            user.email = "user\(i)@example.com"
            user.createdAt = Date()
        }
        
        // Save background context
        do {
            try context.save()
        } catch {
            print("Error saving background context: \(error)")
        }
    }
}
```

### Batch Insert

```swift
func batchInsertUsers(count: Int) {
    let context = CoreDataStack.shared.persistentContainer.newBackgroundContext()
    
    context.performAndWait {
        for i in 0..<count {
            let user = User(context: context)
            user.id = UUID()
            user.name = "User \(i)"
            user.email = "user\(i)@example.com"
            user.createdAt = Date()
            
            // Save every 1000 objects
            if i % 1000 == 0 {
                try? context.save()
                context.reset()
            }
        }
        
        try? context.save()
    }
}
```

### Batch Update

```swift
func batchUpdateUsers() {
    let context = CoreDataStack.shared.context
    
    let batchUpdate = NSBatchUpdateRequest(entityName: "User")
    batchUpdate.propertiesToUpdate = ["email": "updated@example.com"]
    batchUpdate.resultType = .updatedObjectsCountResultType
    
    do {
        let result = try context.execute(batchUpdate) as? NSBatchUpdateResult
        let count = result?.result as? Int
        print("Updated \(count ?? 0) users")
    } catch {
        print("Error: \(error)")
    }
}
```

---

## Migration

### Lightweight Migration

```swift
lazy var persistentContainer: NSPersistentContainer = {
    let container = NSPersistentContainer(name: "MyApp")
    
    let storeDescription = container.persistentStoreDescriptions.first
    storeDescription?.shouldMigrateStoreAutomatically = true
    storeDescription?.shouldInferMappingModelAutomatically = true
    
    container.loadPersistentStores { description, error in
        if let error = error {
            fatalError("Unable to load persistent stores: \(error)")
        }
    }
    
    return container
}()
```

### Manual Migration

```swift
class MigrationManager {
    static func migrateStoreIfNeeded() {
        let storeURL = NSPersistentContainer.defaultDirectoryURL()
            .appendingPathComponent("MyApp.sqlite")
        
        guard FileManager.default.fileExists(atPath: storeURL.path) else {
            return
        }
        
        do {
            let metadata = try NSPersistentStoreCoordinator.metadataForPersistentStore(
                ofType: NSSQLiteStoreType,
                at: storeURL,
                options: nil
            )
            
            guard let modelURL = Bundle.main.url(forResource: "MyApp", withExtension: "momd"),
                  let model = NSManagedObjectModel(contentsOf: modelURL) else {
                return
            }
            
            if !model.isConfiguration(withName: nil, compatibleWithStoreMetadata: metadata) {
                print("Migration needed")
                // Perform migration
            }
        } catch {
            print("Error checking migration: \(error)")
        }
    }
}
```

---

## Best Practices

### 1. Use Background Context for Heavy Operations

```swift
// ✅ Bueno - Background context
func importLargeDataset() {
    let context = CoreDataStack.shared.persistentContainer.newBackgroundContext()
    
    context.perform {
        // Heavy operations
        try? context.save()
    }
}
```

### 2. Batch Operations for Multiple Changes

```swift
// ✅ Bueno - Batch delete
let fetchRequest: NSFetchRequest<NSFetchRequestResult> = User.fetchRequest()
let batchDelete = NSBatchDeleteRequest(fetchRequest: fetchRequest)

try? context.execute(batchDelete)
```

### 3. Use Fetch Request Templates

```swift
// In .xcdatamodeld, create fetch request template
// Then use it:
let request = managedObjectModel.fetchRequestTemplate(forName: "ActiveUsers")
```

### 4. Faulting para Performance

```swift
// Fetch solo IDs
let request = User.fetchRequest()
request.resultType = .managedObjectIDResultType

// O fetch specific properties
request.propertiesToFetch = ["name", "email"]
```

---

## Checklist de Aprendizaje

### Básico
- [ ] Setup CoreData stack
- [ ] Create entities
- [ ] CRUD operations
- [ ] Simple fetch requests

### Intermedio
- [ ] Predicates
- [ ] Sort descriptors
- [ ] Relationships
- [ ] NSFetchedResultsController

### Avanzado
- [ ] Background contexts
- [ ] Batch operations
- [ ] Migration
- [ ] Performance optimization

---

## Próximos Pasos

### 1. Advanced Topics
- Cloud sync con CloudKit
- CoreData + Combine
- Custom migrations

### 2. Alternatives
- SwiftData (iOS 17+)
- Realm
- SQLite.swift

---

## Recursos

### Documentación
- 📚 [CoreData](https://developer.apple.com/documentation/coredata)
- 📖 [CoreData Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/)

### Videos
- 🎥 [WWDC: CoreData](https://developer.apple.com/videos/play/wwdc2019/230/)
- 🎥 [What's New in CoreData](https://developer.apple.com/videos/play/wwdc2021/10017/)

### Artículos
- 📝 [Ray Wenderlich CoreData](https://www.raywenderlich.com/7569-getting-started-with-core-data-tutorial)
- 📝 [Hacking with Swift](https://www.hackingwithswift.com/quick-start/swiftui/introduction-to-core-data)

---

**Anterior:** [URLSession](../09-Networking/URLSession.md)  
**Próximo:** [XCTest](../10-Testing/XCTest.md)
