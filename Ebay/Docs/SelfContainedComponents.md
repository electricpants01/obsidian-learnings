# Self-Contained Components

Some complex UI sections follow a factory pattern that encapsulates their entire architecture — ViewModel, state, effects, navigation — as a reusable drop-in component.

---

## Pattern Overview

```
ComponentFactory (interface)
  │
  ▼
ComponentFactoryImpl (implementation)
  ├── Creates ViewModel (via Dagger)
  ├── Provides Composables (HostedComponent())
  └── Handles navigation callbacks
```

### Interface

```kotlin
interface MyComponentFactory {
    @Composable
    fun HostedComponent(
        modifier: Modifier = Modifier,
        input: MyInputData,
        onNavigate: (MyNavigationAction) -> Unit
    )
}
```

### Usage

```kotlin
// In a screen composable
MyComponentFactory.HostedComponent(
    input = data,
    onNavigate = { action ->
        when (action) {
            is MyNavigationAction.ViewDetails -> navController.navigate(...)
        }
    }
)
```

### Screen vs Component Architecture

| Aspect | Screen | Self-Contained Component |
|---|---|---|
| Route | `@Serializable` data class | Factory interface |
| Navigation | `NavGraphBuilder` extension | Navigation callbacks |
| ViewModel creation | `viewModel(factory=...)` | Factory-owned |
| Preview | Theme wrapper | Preview factory |
| State sharing | Via nav args | Via component callbacks |

---

## Existing Components

| Component | Factory | Feature |
|---|---|---|
| **Unified Price Guidance** | `UnifiedPriceGuidanceFactory` | Market data, price insights tabs |
| **Graded Card Display** | `ShowGradedCardFactory` | Card Insights (PSA, BGS, etc.) |
| **Condition Selection** | `ConditionSelectionFactory` | Graded/ungraded condition picker |
| **Photo Carousel** | `PhotoCarouselFactory` | Image carousel for item photos |

---

## When to Use

Use a self-contained component when:

- The UI section has its own ViewModel with independent state
- It needs to be reused across multiple screens
- It has complex internal navigation that shouldn't leak to the host
- It can be tested independently

Otherwise, keep it inline in the screen's ViewModel and composables.
