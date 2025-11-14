# Lists & Forms en SwiftUI - Guía Completa

## Tabla de Contenido
1. [List Básica](#list-básica)
2. [List con Sections](#list-con-sections)
3. [List Styles](#list-styles)
4. [Swipe Actions](#swipe-actions)
5. [Edit Mode](#edit-mode)
6. [Search](#search)
7. [Forms](#forms)
8. [Form Controls](#form-controls)
9. [Validation](#validation)
10. [Checklist de Aprendizaje](#checklist-de-aprendizaje)
11. [Próximos Pasos](#próximos-pasos)
12. [Recursos](#recursos)

---

## List Básica

### List con Arrays

```swift
struct BasicListView: View {
    let items = ["Item 1", "Item 2", "Item 3", "Item 4", "Item 5"]
    
    var body: some View {
        List(items, id: \.self) { item in
            Text(item)
        }
    }
}
```

### List con Identifiable

```swift
struct User: Identifiable {
    let id = UUID()
    let name: String
    let email: String
    let avatar: String
}

struct UserListView: View {
    let users = [
        User(name: "Ana García", email: "ana@example.com", avatar: "person.circle"),
        User(name: "Luis Rodríguez", email: "luis@example.com", avatar: "person.circle.fill"),
        User(name: "María López", email: "maria@example.com", avatar: "person.circle")
    ]
    
    var body: some View {
        NavigationStack {
            List(users) { user in
                UserRow(user: user)
            }
            .navigationTitle("Users")
        }
    }
}

struct UserRow: View {
    let user: User
    
    var body: some View {
        HStack(spacing: 15) {
            Image(systemName: user.avatar)
                .font(.title)
                .foregroundColor(.blue)
                .frame(width: 50)
            
            VStack(alignment: .leading, spacing: 5) {
                Text(user.name)
                    .font(.headline)
                
                Text(user.email)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
        .padding(.vertical, 5)
    }
}
```

### List con Ranges

```swift
struct RangeListView: View {
    var body: some View {
        List(1...100, id: \.self) { number in
            Text("Row \(number)")
        }
    }
}
```

### Dynamic List

```swift
struct DynamicListView: View {
    @State private var items = ["Initial Item"]
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(items, id: \.self) { item in
                    Text(item)
                }
            }
            .navigationTitle("Dynamic List")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("Add") {
                        items.append("Item \(items.count + 1)")
                    }
                }
            }
        }
    }
}
```

---

## List con Sections

### Sections Básicas

```swift
struct SectionedListView: View {
    var body: some View {
        List {
            Section("Favorites") {
                Text("Item 1")
                Text("Item 2")
                Text("Item 3")
            }
            
            Section("Recent") {
                Text("Item 4")
                Text("Item 5")
            }
            
            Section("Archive") {
                Text("Item 6")
                Text("Item 7")
            }
        }
    }
}
```

### Sections con Headers y Footers

```swift
struct DetailedSectionsView: View {
    var body: some View {
        List {
            Section {
                Text("Setting 1")
                Text("Setting 2")
            } header: {
                Text("General")
            } footer: {
                Text("These are general settings")
            }
            
            Section {
                Toggle("Notifications", isOn: .constant(true))
                Toggle("Dark Mode", isOn: .constant(false))
            } header: {
                Text("Preferences")
            } footer: {
                Text("Customize your experience")
            }
        }
    }
}
```

### Sections Dinámicas

```swift
struct Category: Identifiable {
    let id = UUID()
    let name: String
    let items: [String]
}

struct DynamicSectionsView: View {
    let categories = [
        Category(name: "Fruits", items: ["Apple", "Banana", "Orange"]),
        Category(name: "Vegetables", items: ["Carrot", "Broccoli", "Spinach"]),
        Category(name: "Grains", items: ["Rice", "Wheat", "Oats"])
    ]
    
    var body: some View {
        List {
            ForEach(categories) { category in
                Section(category.name) {
                    ForEach(category.items, id: \.self) { item in
                        Text(item)
                    }
                }
            }
        }
    }
}
```

---

## List Styles

### Disponibles

```swift
struct ListStylesView: View {
    var body: some View {
        VStack {
            // Automatic (default)
            List {
                Text("Automatic")
            }
            .listStyle(.automatic)
            
            // Plain
            List {
                Text("Plain")
            }
            .listStyle(.plain)
            
            // Grouped
            List {
                Text("Grouped")
            }
            .listStyle(.grouped)
            
            // Inset Grouped
            List {
                Text("Inset Grouped")
            }
            .listStyle(.insetGrouped)
            
            // Sidebar (iPad)
            List {
                Text("Sidebar")
            }
            .listStyle(.sidebar)
        }
    }
}
```

### Row Separators

```swift
struct SeparatorStyleView: View {
    var body: some View {
        List {
            Text("Visible Separator")
            
            Text("Hidden Separator")
                .listRowSeparator(.hidden)
            
            Text("Tinted Separator")
                .listRowSeparatorTint(.blue)
        }
    }
}
```

### Row Insets

```swift
struct InsetView: View {
    var body: some View {
        List {
            Text("Default Inset")
            
            Text("Custom Inset")
                .listRowInsets(EdgeInsets(top: 10, leading: 50, bottom: 10, trailing: 20))
            
            Text("No Inset")
                .listRowInsets(EdgeInsets())
        }
    }
}
```

---

## Swipe Actions

### Swipe Actions Básicas

```swift
struct SwipeActionsView: View {
    @State private var items = ["Item 1", "Item 2", "Item 3", "Item 4"]
    
    var body: some View {
        List {
            ForEach(items, id: \.self) { item in
                Text(item)
                    .swipeActions {
                        Button(role: .destructive) {
                            if let index = items.firstIndex(of: item) {
                                items.remove(at: index)
                            }
                        } label: {
                            Label("Delete", systemImage: "trash")
                        }
                    }
            }
        }
    }
}
```

### Múltiples Swipe Actions

```swift
struct MultipleSwipeView: View {
    @State private var items = ["Email 1", "Email 2", "Email 3"]
    
    var body: some View {
        List {
            ForEach(items, id: \.self) { item in
                Text(item)
                    .swipeActions(edge: .leading) {
                        Button {
                            print("Marked as read")
                        } label: {
                            Label("Read", systemImage: "envelope.open")
                        }
                        .tint(.blue)
                    }
                    .swipeActions(edge: .trailing) {
                        Button(role: .destructive) {
                            if let index = items.firstIndex(of: item) {
                                items.remove(at: index)
                            }
                        } label: {
                            Label("Delete", systemImage: "trash")
                        }
                        
                        Button {
                            print("Flagged")
                        } label: {
                            Label("Flag", systemImage: "flag")
                        }
                        .tint(.orange)
                    }
            }
        }
    }
}
```

### Full Swipe

```swift
struct FullSwipeView: View {
    @State private var items = ["Task 1", "Task 2", "Task 3"]
    
    var body: some View {
        List {
            ForEach(items, id: \.self) { item in
                Text(item)
                    .swipeActions(edge: .trailing, allowsFullSwipe: true) {
                        Button(role: .destructive) {
                            if let index = items.firstIndex(of: item) {
                                items.remove(at: index)
                            }
                        } label: {
                            Label("Delete", systemImage: "trash")
                        }
                    }
            }
        }
    }
}
```

---

## Edit Mode

### Delete y Move

```swift
struct EditableListView: View {
    @State private var items = ["Item 1", "Item 2", "Item 3", "Item 4", "Item 5"]
    @State private var editMode: EditMode = .inactive
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(items, id: \.self) { item in
                    Text(item)
                }
                .onDelete { indexSet in
                    items.remove(atOffsets: indexSet)
                }
                .onMove { source, destination in
                    items.move(fromOffsets: source, toOffset: destination)
                }
            }
            .navigationTitle("Editable List")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    EditButton()
                }
                
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("Add") {
                        items.insert("New Item", at: 0)
                    }
                }
            }
            .environment(\.editMode, $editMode)
        }
    }
}
```

### Custom Edit Actions

```swift
struct CustomEditView: View {
    @State private var todos = [
        Todo(title: "Buy groceries", isCompleted: false),
        Todo(title: "Write code", isCompleted: true),
        Todo(title: "Exercise", isCompleted: false)
    ]
    
    var body: some View {
        NavigationStack {
            List {
                ForEach($todos) { $todo in
                    HStack {
                        Image(systemName: todo.isCompleted ? "checkmark.circle.fill" : "circle")
                            .foregroundColor(todo.isCompleted ? .green : .gray)
                            .onTapGesture {
                                todo.isCompleted.toggle()
                            }
                        
                        Text(todo.title)
                            .strikethrough(todo.isCompleted)
                    }
                }
                .onDelete { indexSet in
                    todos.remove(atOffsets: indexSet)
                }
            }
            .navigationTitle("Todos")
            .toolbar {
                EditButton()
            }
        }
    }
}

struct Todo: Identifiable {
    let id = UUID()
    var title: String
    var isCompleted: Bool
}
```

---

## Search

### Searchable

```swift
struct SearchableListView: View {
    @State private var searchText = ""
    
    let allItems = ["Apple", "Banana", "Cherry", "Date", "Elderberry", "Fig", "Grape"]
    
    var filteredItems: [String] {
        if searchText.isEmpty {
            return allItems
        } else {
            return allItems.filter { $0.localizedCaseInsensitiveContains(searchText) }
        }
    }
    
    var body: some View {
        NavigationStack {
            List(filteredItems, id: \.self) { item in
                Text(item)
            }
            .searchable(text: $searchText, prompt: "Search fruits")
            .navigationTitle("Fruits")
        }
    }
}
```

### Search con Scopes

```swift
struct SearchScopesView: View {
    @State private var searchText = ""
    @State private var selectedScope = SearchScope.all
    
    enum SearchScope {
        case all, active, completed
    }
    
    let tasks = [
        Task(title: "Task 1", isCompleted: false),
        Task(title: "Task 2", isCompleted: true),
        Task(title: "Task 3", isCompleted: false)
    ]
    
    var filteredTasks: [Task] {
        tasks.filter { task in
            let matchesSearch = searchText.isEmpty || task.title.localizedCaseInsensitiveContains(searchText)
            let matchesScope: Bool
            
            switch selectedScope {
            case .all:
                matchesScope = true
            case .active:
                matchesScope = !task.isCompleted
            case .completed:
                matchesScope = task.isCompleted
            }
            
            return matchesSearch && matchesScope
        }
    }
    
    var body: some View {
        NavigationStack {
            List(filteredTasks) { task in
                HStack {
                    Text(task.title)
                    Spacer()
                    if task.isCompleted {
                        Image(systemName: "checkmark")
                            .foregroundColor(.green)
                    }
                }
            }
            .searchable(text: $searchText) {
                Text("All").tag(SearchScope.all)
                Text("Active").tag(SearchScope.active)
                Text("Completed").tag(SearchScope.completed)
            }
            .navigationTitle("Tasks")
        }
    }
}

struct Task: Identifiable {
    let id = UUID()
    let title: String
    let isCompleted: Bool
}
```

---

## Forms

### Form Básica

```swift
struct BasicFormView: View {
    @State private var name = ""
    @State private var email = ""
    @State private var age = 18
    @State private var isSubscribed = false
    
    var body: some View {
        NavigationStack {
            Form {
                Section("Personal Information") {
                    TextField("Name", text: $name)
                    TextField("Email", text: $email)
                        .textContentType(.emailAddress)
                        .keyboardType(.emailAddress)
                    
                    Stepper("Age: \(age)", value: $age, in: 0...120)
                }
                
                Section("Preferences") {
                    Toggle("Newsletter", isOn: $isSubscribed)
                }
                
                Section {
                    Button("Submit") {
                        submitForm()
                    }
                }
            }
            .navigationTitle("Sign Up")
        }
    }
    
    func submitForm() {
        print("Name: \(name), Email: \(email)")
    }
}
```

### Form Completa

```swift
struct CompleteFormView: View {
    @State private var username = ""
    @State private var email = ""
    @State private var password = ""
    @State private var confirmPassword = ""
    @State private var birthdate = Date()
    @State private var country = "USA"
    @State private var agreeToTerms = false
    
    let countries = ["USA", "UK", "Canada", "Australia"]
    
    var body: some View {
        NavigationStack {
            Form {
                Section("Account") {
                    TextField("Username", text: $username)
                        .textContentType(.username)
                        .autocapitalization(.none)
                    
                    TextField("Email", text: $email)
                        .textContentType(.emailAddress)
                        .keyboardType(.emailAddress)
                        .autocapitalization(.none)
                    
                    SecureField("Password", text: $password)
                        .textContentType(.newPassword)
                    
                    SecureField("Confirm Password", text: $confirmPassword)
                        .textContentType(.newPassword)
                }
                
                Section("Personal") {
                    DatePicker("Birthdate", selection: $birthdate, displayedComponents: .date)
                    
                    Picker("Country", selection: $country) {
                        ForEach(countries, id: \.self) { country in
                            Text(country)
                        }
                    }
                }
                
                Section {
                    Toggle("I agree to the terms", isOn: $agreeToTerms)
                }
                
                Section {
                    Button("Create Account") {
                        createAccount()
                    }
                    .disabled(!isFormValid)
                }
            }
            .navigationTitle("Sign Up")
        }
    }
    
    var isFormValid: Bool {
        !username.isEmpty &&
        !email.isEmpty &&
        password.count >= 6 &&
        password == confirmPassword &&
        agreeToTerms
    }
    
    func createAccount() {
        print("Account created")
    }
}
```

---

## Form Controls

### Pickers

```swift
struct PickerFormView: View {
    @State private var selectedColor = "Red"
    @State private var selectedSize = Size.medium
    
    enum Size: String, CaseIterable {
        case small = "Small"
        case medium = "Medium"
        case large = "Large"
    }
    
    let colors = ["Red", "Green", "Blue", "Yellow"]
    
    var body: some View {
        Form {
            Section("Color") {
                Picker("Select Color", selection: $selectedColor) {
                    ForEach(colors, id: \.self) { color in
                        Text(color)
                    }
                }
            }
            
            Section("Size") {
                Picker("Select Size", selection: $selectedSize) {
                    ForEach(Size.allCases, id: \.self) { size in
                        Text(size.rawValue)
                    }
                }
                .pickerStyle(.segmented)
            }
        }
    }
}
```

### DatePicker & ColorPicker

```swift
struct SpecialPickersView: View {
    @State private var selectedDate = Date()
    @State private var selectedColor = Color.blue
    
    var body: some View {
        Form {
            Section("Date & Time") {
                DatePicker("Select Date", selection: $selectedDate)
                
                DatePicker("Date Only", selection: $selectedDate, displayedComponents: .date)
                
                DatePicker("Time Only", selection: $selectedDate, displayedComponents: .hourAndMinute)
            }
            
            Section("Color") {
                ColorPicker("Select Color", selection: $selectedColor)
            }
        }
    }
}
```

---

## Validation

### Form Validation

```swift
struct ValidatedFormView: View {
    @State private var email = ""
    @State private var password = ""
    @State private var showError = false
    
    var isValidEmail: Bool {
        email.contains("@") && email.contains(".")
    }
    
    var isValidPassword: Bool {
        password.count >= 6
    }
    
    var body: some View {
        Form {
            Section {
                TextField("Email", text: $email)
                    .textContentType(.emailAddress)
                    .autocapitalization(.none)
                
                if !email.isEmpty && !isValidEmail {
                    Text("Invalid email format")
                        .font(.caption)
                        .foregroundColor(.red)
                }
            } header: {
                Text("Email")
            }
            
            Section {
                SecureField("Password", text: $password)
                
                if !password.isEmpty && !isValidPassword {
                    Text("Password must be at least 6 characters")
                        .font(.caption)
                        .foregroundColor(.red)
                }
            } header: {
                Text("Password")
            }
            
            Section {
                Button("Login") {
                    login()
                }
                .disabled(!isValidEmail || !isValidPassword)
            }
        }
    }
    
    func login() {
        print("Login successful")
    }
}
```

---

## Checklist de Aprendizaje

### Básico
- [ ] Crear Lists básicas
- [ ] Usar ForEach con Identifiable
- [ ] Sections en Lists
- [ ] Swipe actions básicas

### Intermedio
- [ ] Edit mode (delete, move)
- [ ] Search con searchable
- [ ] Forms completas
- [ ] List styles
- [ ] Form validation

### Avanzado
- [ ] Custom List rows
- [ ] Search con scopes
- [ ] Complex forms
- [ ] Performance con LazyVStack
- [ ] Custom swipe actions

---

## Próximos Pasos

### 1. Performance
- LazyVStack vs List
- Pagination
- Prefetching

### 2. Advanced Features
- Pull to refresh
- Context menus
- Custom list backgrounds

---

## Recursos

### Documentación
- 📚 [List](https://developer.apple.com/documentation/swiftui/list)
- 📖 [Form](https://developer.apple.com/documentation/swiftui/form)
- 📘 [Searchable](https://developer.apple.com/documentation/swiftui/view/searchable(text:placement:))

### Videos
- 🎥 [Lists in SwiftUI](https://www.youtube.com/watch?v=3krC2c56ceQ)
- 🎥 [Forms Tutorial](https://www.youtube.com/watch?v=4GjXq2Sr55Q)

### Artículos
- 📝 [Hacking with Swift - Lists](https://www.hackingwithswift.com/quick-start/swiftui/working-with-lists)
- 📝 [Swift by Sundell - Forms](https://www.swiftbysundell.com/articles/swiftui-forms/)

---

**Anterior:** [Views & Modifiers](Views-Modifiers.md)  
**Próximo:** [Animations](Animations.md)
