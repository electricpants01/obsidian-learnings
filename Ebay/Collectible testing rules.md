  
  
  *Applies to: digitalCollections module only*  
  
## Scope  
  
These testing rules apply exclusively to code within the `digitalCollections/` directory and its subdirectories. These rules supplement the testing guidelines in `digitalCollections/docs/CollectiblesArchitectureAndBestPractices.md` and should be used in conjunction with the broader architectural patterns established in that document.  
  [[Collectibles Architecture and Best Practices]]
## Core Testing Requirements  
  
### Assertion Library  
- **Prefer `shouldBe`** for equality assertions  
- **Import**: `com.ebay.mobile.test.support.hamcrest.shouldBe`  
- **Prefer `assertThat`** for all other assertions  
- **Import**: `org.hamcrest.MatcherAssert.assertThat`  
  
```kotlin  
// Preferred for equality assertions  
result shouldBe expectedValue  
filters shouldBe PriceGuidanceFilters(  
    listingFormats = SalesFormatFilter.ALL,    timeFrame = TimeFrame.THREE_MONTHS,    recentSalesSort = RecentSalesSortOption.DATE_LAST_SOLD_DESC)  
  
// Preferred for other assertions  
assertThat(result, hasSize(3))  
assertThat(result, containsInAnyOrder(item1, item2, item3))  
assertThat(result, not(nullValue()))  
  
// Avoid using assert keyword  
assert(result == expectedValue)  
```  
  
### Mocking Strategy  
- **Use mockk for all new test classes**  
- **When modifying existing test classes**, use whichever mocking library is currently in use in that class  
- Prefer fakes over mocks when possible (as outlined in the architecture document)  
  
```kotlin  
// New test classes  
private val mockActiveListingsUseCase = mockk<ActiveListingsUseCase>()  
  
// Existing test class already using Mockito - keep using Mockito  
@Mock  
private lateinit var existingMockRepository: ExistingRepository  
```  
  
### Data Object Testing Guidelines  
- **Do NOT write tests** for data objects or sealed classes unless there is specific logic within them that needs to be tested  
- **Do NOT write tests** like:  
  - "copy function works correctly"  
  - "equals and hashCode work correctly"  
  - Basic data class functionality  
- **You MAY write tests** that verify:  
  - Default values are correct  
  - Custom logic within data classes  
  - Validation logic  
  - Computed properties  
  
```kotlin  
// Acceptable - testing default values  
@Test  
fun `default constructor creates filters with expected default values`() {  
    val filters = PriceGuidanceFilters()        filters shouldBe PriceGuidanceFilters(  
        listingFormats = SalesFormatFilter.ALL,        timeFrame = TimeFrame.THREE_MONTHS,        recentSalesSort = RecentSalesSortOption.DATE_LAST_SOLD_DESC,        condition = null,        graderGradeFilter = null    )}  
  
// Avoid - testing basic data class functionality  
@Test  
fun `copy function works correctly`() {  
    val original = PriceGuidanceFilters()    val copied = original.copy(timeFrame = TimeFrame.ONE_MONTH)    // This is unnecessary}  
```  
  
### Flow Testing  
- **Use turbine** when testing code that involves Kotlin flows  
- **Encourage `expectMostRecentItem()`** when you only need to verify the latest emission  
- Use `awaitItem()` when you need to verify the sequence of emissions  
- Turbine provides clean, readable flow testing capabilities  
  
```kotlin  
// Use expectMostRecentItem() for latest value verification  
@Test  
fun `repository emits latest data after refresh`() = runTest {  
    repository.dataFlow.test {        repository.refresh()                // Skip loading states and get the final result  
        expectMostRecentItem() shouldBe SuccessState(expectedData)    }}  
  
// Use awaitItem() for sequential verification  
@Test  
fun `repository emits loading then success states`() = runTest {  
    repository.dataFlow.test {        repository.refresh()                awaitItem() shouldBe LoadingState  
        awaitItem() shouldBe SuccessState(expectedData)        awaitComplete()    }}  
```  
  
### Test Data Management  
- **Minimize duplication** of dummy data in tests when possible  
- Create reusable dummy data objects that can be shared across multiple tests  
- Use fixtures to reduce the need for dummy data objects  
- Create reusable fakes rather than unique instances for each test class  
  
```kotlin  
// Reusable test data  
object TestData {  
    val defaultPriceGuidanceInput = ListingPriceGuidanceInput(        listingId = "listing123",        productId = "product456",        categoryId = "category789",        title = "Test Item",        condition = ItemCondition.NEW,        filters = PriceGuidanceFilters()    )        val mockListingItems = listOf<ListingItem>()  
}  
  
// Usage in tests  
@Test  
fun `test with reusable data`() {  
    val input = TestData.defaultPriceGuidanceInput.copy(listingId = "different123")    // Use input in test}  
```  
  
## Architecture-Aligned Testing Patterns  
  
### UDF (Unidirectional Data Flow) Testing  
  
When testing ViewModels that use `CollectiblesScaffold`, follow these patterns:  
  
#### Testing Event Handling  
```kotlin  
@Test  
fun `onEvent handles UpdateCurrentImageIndex correctly`() = runTest {  
    // Given    val viewModel = createViewModel()    val expectedIndex = 2        // When  
    viewModel.onEvent(CollectibleItemDetailsEvent.UpdateCurrentImageIndex(expectedIndex))        // Then  
    viewModel.viewState.value.collectibleItemUiState.currentPhotoIndex shouldBe expectedIndex}  
```  
  
#### Testing Side Effects  
```kotlin  
@Test  
fun `toggleBookmark emits ShowSnackBar effect on error`() = runTest {  
    // Given    val viewModel = createViewModel()    coEvery { bookmarkingUseCase.toggleBookmark(any(), any()) } returns        flowOf(BookmarkingResult.Error("Network error"))  
        // When  
    viewModel.onEvent(CollectibleItemDetailsEvent.ToggleBookmark(true))        // Then  
    viewModel.sideEffects.test {  
        expectMostRecentItem() shouldBe CollectibleItemDetailsEffect.ShowSnackBar(            UiText.StringResource(R.string.dc_folders_general_error)        )    }  
}  
```  
  
#### Testing Reactive Flows and Initialization  
```kotlin  
@Test  
fun `initContent triggers data refresh`() = runTest {  
    // Given    val mockRepository = mockk<DataRepository>()    coEvery { mockRepository.fetchData() } returns flowOf(expectedData)    val viewModel = createViewModel(repository = mockRepository)        // When  
    viewModel.onEvent(MyScreenEvent.InitContent(testInput))        // Then  
    coVerify { mockRepository.fetchData() }  
    viewModel.viewState.test {  
        expectMostRecentItem().data shouldBe expectedData    }  
}  
```  
  
### Component Testing  
  
#### Testing Self-Contained Components  
```kotlin  
class ConditionSelectionComponentTest {  
        @get:Rule  
    val composeTestRule = createComposeRule()        private val conditionSelectionPageObject = ConditionSelectionPageObject(composeTestRule)  
        @Test  
    fun `component factory creates component with correct input`() {        // Given        val input = ConditionSelectionInput(            categoryId = "123",            itemConditionModel = null        )                // When  
        composeTestRule.setContent {            conditionSelectionFactory.HostedComponent(                modifier = Modifier,                viewModelStoreOwner = mockViewModelStoreOwner,                input = input,                showEntryPointField = true,                isSheetVisible = false,                onDismiss = {},                onSelectionChanged = {}            )        }                // Then - Use page object for assertions  
        conditionSelectionPageObject.conditionField().assertExists()        conditionSelectionPageObject.conditionFieldLabel().assertTextEquals("Condition")    }}  
```  
  
#### Testing Component Callbacks  
```kotlin  
@Test  
fun `component calls onSelectionChanged when selection is made`() = runTest {  
    // Given    var receivedSelection: ItemConditionModel? = null    val expectedSelection = ItemConditionModel.NEW        // When  
    composeTestRule.setContent {  
        conditionSelectionFactory.HostedComponent(            // ... other parameters            onSelectionChanged = { selection ->  
                receivedSelection = selection            }  
        )    }  
        // Simulate selection using page object  
    conditionSelectionPageObject.conditionField().performClick()    conditionSelectionPageObject.newConditionOption().performClick()    conditionSelectionPageObject.confirmButton().performClick()        // Then  
    receivedSelection shouldBe expectedSelection}  
```  
  
### UI Testing with Page Object Pattern  
  
Follow the Page Object Pattern as outlined in the architecture document:  
  
```kotlin  
class CollectibleLibraryPageObject(private val composeTestRule: ComposeTestRule) {  
        fun folderMaxNotice(): SemanticsNodeInteraction = composeTestRule  
        .onNodeWithText("Folder limit reached. Delete a folder to create a new one.", useUnmergedTree = true)        fun folderListItemCount(count: Int): SemanticsNodeInteraction = composeTestRule  
        .onNodeWithText("($count)", useUnmergedTree = true)}  
  
// Usage in tests  
@Test  
fun `displays folder limit notice when maximum folders reached`() {  
    // Given    setupMaximumFolders()        // When  
    launchCollectionActivity()        // Then  
    collectibleLibraryPageObject.folderMaxNotice().assertExists()}  
```  
  
## Testing Best Practices Summary  
  
### DO  
- Use `shouldBe` for equality assertions  
- Use `assertThat` for other assertions  
- Use mockk for new test classes  
- Test default values in data objects when meaningful  
- Use turbine for flow testing  
- Use `expectMostRecentItem()` when you only need the latest emission  
- Create reusable test data and fakes  
- Follow UDF patterns when testing ViewModels  
- Use Page Object Pattern for UI tests  
- Test component integration through factory interfaces  
- Prefer Robolectric UI tests over instrumented tests  
  
### DON'T  
- Use the `assert` keyword  
- Test basic data class functionality (copy, equals, hashCode)  
- Create duplicate dummy data across test classes  
- Mix mocking libraries within the same test class  
- Test UI components by directly accessing ViewModels  
- Write assertions in Page Object Models  
- Create event/effect chains in tests  
- Use direct UI matchers in component tests (use page objects instead)  
  
## Integration with Architecture Document  
  
These testing rules work in conjunction with the patterns established in `digitalCollections/docs/CollectiblesArchitectureAndBestPractices.md`. For comprehensive guidance on:  
  
- **Architectural Layering**: Understand how to structure tests across UI, Domain, and Data layers  
- **UDF Implementation**: Learn the Model-View-Intent patterns that inform these testing approaches  
- **Component Architecture**: Understand the factory pattern and self-contained component design  
- **Navigation Testing**: Learn how to test navigation flows and screen integration  
  
Refer to the main architecture document for detailed explanations of the patterns that these testing rules support.  
  
---  
  
*This document ensures consistent, maintainable, and architecture-aligned testing practices within the digitalCollections module.*