# Accessibility

## Content Descriptions

```kotlin
@Composable
fun AccessibleButton() {
    Button(
        onClick = {},
        modifier = Modifier.semantics {
            contentDescription = "Submit button"
        }
    ) {
        Icon(Icons.Default.Check, contentDescription = null)
        Text("Submit")
    }
}
```

## TalkBack

```kotlin
// Announce for accessibility
view.announceForAccessibility("Action completed")
```

## Testing

```kotlin
composeTestRule.onNodeWithContentDescription("Submit button")
    .assertIsDisplayed()
```

## Recursos
- [Accessibility](https://developer.android.com/guide/topics/ui/accessibility)
