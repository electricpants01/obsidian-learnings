# CoreData Relationships

## One-to-Many

```swift
// Author -> Books (one-to-many)
class Author: NSManagedObject {
    @NSManaged var name: String
    @NSManaged var books: Set<Book>
}

class Book: NSManagedObject {
    @NSManaged var title: String
    @NSManaged var author: Author
}
```

## Recursos
- [CoreData](https://developer.apple.com/documentation/coredata)
