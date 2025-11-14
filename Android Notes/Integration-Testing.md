# Integration Testing

## Setup

```kotlin
dependencies {
    androidTestImplementation("androidx.test.ext:junit:1.1.5")
    androidTestImplementation("androidx.test.espresso:espresso-core:3.5.1")
    androidTestImplementation("androidx.compose.ui:ui-test-junit4:1.6.0")
}
```

## Compose Test

```kotlin
@get:Rule
val composeTestRule = createComposeRule()

@Test
fun myTest() {
    composeTestRule.setContent {
        MyScreen()
    }
    
    composeTestRule.onNodeWithText("Hello").assertIsDisplayed()
    composeTestRule.onNodeWithTag("button").performClick()
}
```

## Recursos
- [Testing](https://developer.android.com/training/testing)
