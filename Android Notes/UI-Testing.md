# UI Testing

## Compose Testing

```kotlin
@get:Rule
val composeTestRule = createComposeRule()

@Test
fun loginTest() {
    composeTestRule.setContent {
        LoginScreen()
    }
    
    // Find and interact
    composeTestRule.onNodeWithTag("email")
        .performTextInput("test@example.com")
    
    composeTestRule.onNodeWithTag("password")
        .performTextInput("password123")
    
    composeTestRule.onNodeWithText("Login")
        .performClick()
    
    // Verify
    composeTestRule.onNodeWithText("Welcome")
        .assertIsDisplayed()
}
```

## Semantics

```kotlin
@Composable
fun MyButton() {
    Button(
        onClick = {},
        modifier = Modifier.testTag("my_button")
    ) {
        Text("Click me")
    }
}
```

## Recursos
- [Compose Testing](https://developer.android.com/jetpack/compose/testing)
