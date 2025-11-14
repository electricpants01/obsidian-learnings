# XCTest - Testing en iOS

## Unit Tests

```swift
import XCTest
@testable import MyApp

class ViewModelTests: XCTestCase {
    var viewModel: UserViewModel!
    
    override func setUp() {
        super.setUp()
        viewModel = UserViewModel()
    }
    
    override func tearDown() {
        viewModel = nil
        super.tearDown()
    }
    
    func testFetchUsers() async throws {
        // Given
        XCTAssertEqual(viewModel.users.count, 0)
        
        // When
        await viewModel.loadUsers()
        
        // Then
        XCTAssertGreaterThan(viewModel.users.count, 0)
    }
    
    func testValidation() {
        // Given
        viewModel.email = "test@example.com"
        viewModel.password = "123456"
        
        // Then
        XCTAssertTrue(viewModel.isValidForm)
    }
}
```

## Mock Objects

```swift
class MockAPIClient: APIClientProtocol {
    var shouldFail = false
    
    func fetchUsers() async throws -> [User] {
        if shouldFail {
            throw NetworkError.serverError
        }
        return [User(id: 1, name: "Test")]
    }
}

func testWithMock() async {
    let mockClient = MockAPIClient()
    let viewModel = UserViewModel(apiClient: mockClient)
    
    await viewModel.loadUsers()
    
    XCTAssertEqual(viewModel.users.count, 1)
}
```

## UI Tests

```swift
class UITests: XCTestCase {
    var app: XCUIApplication!
    
    override func setUp() {
        continueAfterFailure = false
        app = XCUIApplication()
        app.launch()
    }
    
    func testLoginFlow() {
        let emailField = app.textFields["Email"]
        emailField.tap()
        emailField.typeText("test@example.com")
        
        let passwordField = app.secureTextFields["Password"]
        passwordField.tap()
        passwordField.typeText("password")
        
        app.buttons["Login"].tap()
        
        XCTAssertTrue(app.staticTexts["Welcome"].exists)
    }
}
```

## Recursos

- 📚 [XCTest](https://developer.apple.com/documentation/xctest)
- 🎥 [iOS Testing](https://www.youtube.com/watch?v=1C5c3xXPRmo)
