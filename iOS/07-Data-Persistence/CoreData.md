# CoreData - Persistencia de Datos

## Setup

```swift
import CoreData

class PersistenceController {
    static let shared = PersistenceController()
    
    let container: NSPersistentContainer
    
    init() {
        container = NSPersistentContainer(name: "Model")
        container.loadPersistentStores { description, error in
            if let error = error {
                fatalError("Core Data failed: \(error)")
            }
        }
    }
}
```

## CRUD Operations

```swift
// Create
func createUser(name: String, email: String) {
    let context = PersistenceController.shared.container.viewContext
    let user = User(context: context)
    user.name = name
    user.email = email
    
    do {
        try context.save()
    } catch {
        print("Error: \(error)")
    }
}

// Read
func fetchUsers() -> [User] {
    let context = PersistenceController.shared.container.viewContext
    let request: NSFetchRequest<User> = User.fetchRequest()
    
    do {
        return try context.fetch(request)
    } catch {
        return []
    }
}

// Update
func updateUser(_ user: User, name: String) {
    let context = PersistenceController.shared.container.viewContext
    user.name = name
    
    try? context.save()
}

// Delete
func deleteUser(_ user: User) {
    let context = PersistenceController.shared.container.viewContext
    context.delete(user)
    
    try? context.save()
}
```

## SwiftUI Integration

```swift
struct ContentView: View {
    @Environment(\.managedObjectContext) private var viewContext
    @FetchRequest(sortDescriptors: [])
    private var users: FetchedResults<User>
    
    var body: some View {
        List(users) { user in
            Text(user.name ?? "")
        }
    }
}
```

## Recursos

- 📚 [CoreData](https://developer.apple.com/documentation/coredata)
- 🎥 [CoreData Tutorial](https://www.youtube.com/watch?v=O7u9nYWjvKk)
