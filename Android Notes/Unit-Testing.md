# Unit Testing

## Setup

```kotlin
dependencies {
    testImplementation("junit:junit:4.13.2")
    testImplementation("org.mockito:mockito-core:5.8.0")
    testImplementation("org.mockito.kotlin:mockito-kotlin:5.2.1")
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.7.3")
}
```

## Test Básico

```kotlin
class CalculatorTest {
    @Test
    fun `addition works correctly`() {
        val calculator = Calculator()
        assertEquals(4, calculator.add(2, 2))
    }
}
```

## Mock

```kotlin
class ViewModelTest {
    @Mock
    lateinit var repository: Repository
    
    @Before
    fun setup() {
        MockitoAnnotations.openMocks(this)
    }
    
    @Test
    fun `load data success`() = runTest {
        whenever(repository.getData()).thenReturn(listOf(item1, item2))
        
        val viewModel = MyViewModel(repository)
        viewModel.loadData()
        
        assertEquals(UiState.Success, viewModel.state.value)
    }
}
```

## Recursos
- [Testing](https://developer.android.com/training/testing)
