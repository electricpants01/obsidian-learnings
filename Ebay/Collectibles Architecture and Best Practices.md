  ## Child Pages  
- [Architectural Layering](#architectural-layering)  
- [Collectibles Architecture Overview - Unidirectional Data Flow](#collectibles-architecture-overview---unidirectional-data-flow)  
- [Collectibles Testing Best Practices](#collectibles-testing-best-practices)  
- [Collectibles UI Layer Architecture](#collectibles-ui-layer-architecture)  
  
## Purpose  
  
This page is a guide to understanding and applying a consistent architectural pattern in Collectibles. The guide primarily discusses two architectural concepts:  
  
- Architectural Layering (Data, Domain, and UI layers)  
- Unidirectional Data Flow (UDF)  
  
You can refer to this guide for:  
  
- An explanation of layering, UDF and their benefits  
- Tutorials on implementing these architectural patterns  
- Guidelines/Best Practices for applying the architecture in the digital collections codebase  
- Reference links for more detailed explanations  
- A list of features in the code currently using this architecture  
  
## A Brief But Important Note  
  
This guide is **_NOT EXHAUSTIVE_**. This is an overview of architectural concepts, with a focus on how we approach implementing them in our codebase. Explanations will include reference links when relevant. For a thorough examination of the principles upon which this guide is based, see [Android Developer's Guide to App Architecture](https://developer.android.com/topic/architecture). You are encouraged to read this and other provided links for a more thorough understanding of these practices.  
  
---  
  
# Architectural Layering  
  
## Table of Contents  
- [Architectural Layering Overview](#architectural-layering-overview)  
- [UI Layer](#ui-layer)  
- [Data Layer](#data-layer)  
- [Domain Layer](#domain-layer)  
- [Which layer holds View Models?](#which-layer-holds-view-models)  
- [Useful Links and References](#useful-links-and-references)  
  
## Architectural Layering Overview  
  
Architectural Layering is the principle of organizing code into three layers: UI, Domain, and Data. Starting with the UI layer, each layer depends on the next.  
  
<img src="https://developer.android.com/static/topic/libraries/architecture/images/mad-arch-overview.png" alt="Architecture Overview" width="400">  
  
Each layer has its own responsibilities:  
  
- The UI layer displays the UI. It depends on data derived from the data layer  
- The Data layer is where we store, fetch, and manipulate data. It is made of repositories and data sources.  
- The Domain layer sits _between_ the other layers and encapsulates complex business logic. It's mainly used as a way to handle complexity and reusability.  
  
A deeper examination of each layer follows.  
  
## UI Layer  
  
The UI layer is the _presentation layer_. It is responsible for:  
  
- Listening for user interactions  
- Responding to user interactions  
- Communicating to the user  
  
In our code, the UI layer includes (but is not limited to) all of the following elements:  
  
- Activities  
- Fragments  
- Context  
- Compose views  
- State objects  
- Navigation  
- Animations  
- Accessibility features (ex: talkback)  
- Pretty much all Android framework classes  
  
The following are practices that should be followed as closely as possible when working on the UI layer.  
  
### State Objects and Stateless Composables  
  
When building composable screens, practice the following:  
  
- Seek to make your composables **as stateless as possible**. ViewModels and other external logic classes should never be directly referenced by the children of a screen composable. This practice is known as [state hoisting](https://developer.android.com/develop/ui/compose/state#state-hoisting).  
  
### One Screen → One ViewModel → One Screen UI State  
  
The UI layer works best when each "screen" has its own VM and UI state class. For example `CollectibleItemScreen` is managed by `CollectibleItemViewModel`, which modifies and shares `CollectibleItemViewState`. If you run into a situation where you think you need two view models for one screen, do one of the following instead:  
  
- Make a 2nd screen so that each view model has its own  
- If you're relying on a VM from a different screen just to use some of its internal logic, move that logic to a use case and use it as a shared dependency.  
  
### UI State Should be immutable  
  
Immutability is crucial to ensuring the main thread is free to focus on reading state and updating UI. This is especially important in Compose views, but relevant to all UI. Follow these rules to maintain UI state immutability:  
  
- Avoid using `var` properties in state holder classes. Use data classes with val properties instead.  
- Never modify UI state directly in UI. This breaks the "single source of truth" data principle and leads to bugs. The place that provides the data should also be the place that modifies it (usually the VM).  
- Use the `@Immutable` annotation on UI state holder classes, but **only do so if you have followed the other rules**. This annotation helps improve Compose efficiency when used properly, but can have a negative impact when improperly applied.  
  
### UI logic vs business logic  
  
- UI logic (starting activities, fragment transactions, building intents, anything involving context) should always be done in the UI, not the View Model.  
- Business logic should never be done in the UI. Do it in the ViewModel, domain layer, or data layer.  
  
## Data Layer  
  
The data layer is where we create, read, update, and provide data. The data layer is comprised of **data sources** and **repositories**:  
  
<img src="https://developer.android.com/static/topic/libraries/architecture/images/mad-arch-data-overview.png" alt="Data Layer Overview" width="400">  
  
### Data Sources  
  
Data sources are the lowest piece of the data layer, and therefore the lowest part of the whole architecture. They handle the data operations that render "raw" data. Remember:  
  
- Each data source should be responsible for working with only **_one_** source of data  
- Other layers should _**never**_ access data sources directly.  
- Examples of data sources: Apollo operations, retrofit api functions, room databases, android shared preferences  
  
### Repositories  
  
- Upper layers perform data operations (create, read, update, delete) by interacting with repositories.  
- Repositories perform the business logic necessary to translate "raw" data (like network models and database models) into business models.  
- Repositories can rely on multiple data sources. For example, a "read" operation function could pull data from a network data source and a shared pref, then combine the relevant content into a business model.  
- Repositories are the place to resolve potential data conflicts so that the upper layers can rely on a single source of truth.  
  
### Business Models  
  
The data exposed by the data layer should be in the form of **business models**. A business model is a data class that holds only the information necessary for the needs of the feature. For example, a network response object might provide content we don't need. Rather than passing that network response directly to the upper layers, we should refine that raw data into a business model and pass that.  
  
Note that **Business models are not UI models**. At first it may seem like overkill to have 3 different models (network, business, UI state) but this is actually an important practice. See the example below:  
  
```kotlin  
// Apollo's api models are insane, so instead of showing one here I'm going to show a method that converts  
// CollectibleDetailsQuery.Data (a "raw" apollo api model) into CollectibleItem (a business model). Don't worry  
// about the code here. The point is that passing CollectibleDetailsQuery.Data to the upper layers would be a bad idea,  
// as it would require a lot of work to be done in the UI layer to clean this up.  
  
fun CollectibleDetailsQuery.Data.toCollectibleItem(): CollectibleItem? {  
    return this.collections.collectible?.let {        CollectibleItem(            id = it.id,            title = it.title,            price = currencyAmountOrNull(                it.purchase?.price?.priceFragment?.amount,                it.purchase?.price?.priceFragment?.currency.toString(),            ),            purchaseDate = it.purchase?.date?.let { millis ->                EbayDateUtils.parseIso8601DateTime(millis.toString())            },            aspects = it.aspects?.map { aspect ->                aspect.name to (aspect.values?.joinToString(" - ") ?: "-")            }?.toList().orEmpty(),            photoUrls = it.photos?.mapNotNull { photo -> photo.URL }?.toList().orEmpty(),            listingId = it.listingInfo?.lastKnownCollectibleListingDetails?.listing?.id,            draftId = it.listingInfo?.lastKnownCollectibleListingDetails?.draftId,            isArchived = it.listingInfo?.status?.rawValue == "SOLD",            folders = it.folders?.map { folder -> folder.id }.orEmpty(),        )    }}  
  
// This is a business model derived from the raw apollo model. We emit business models to upper layers via our repository  
// functions. However, notice that "price" is a CurrencyAmount. We aren't converting it to a formatted string because  
// different features may need to display the price in different formats or use that data for different purposes.  
  
data class CollectibleItem(  
    val id: String,    val title: String,    val price: CurrencyAmount? = null,    val purchaseDate: Date? = null,    val aspects: List<Pair<String, String>> = emptyList(),    val photoUrls: List<String> = emptyList(),    val vaultInfo: VaultInfo? = null,    val listingId: String? = null,    val draftId: String? = null,    val isArchived: Boolean = false,    val folders: List<String> = emptyList(),)  
  
// Finally, here is a UI state model derived from CollectibleItem. The view model fetches a business model from  
// the repository, then its responsible for converting that data into a state model specifically designed to give  
// the UI exactly the data it needs. For example, the VM would take care of formatting the CurrentyAmount into a string,  
// instead of doing that work directly in the composable.  
  
@Immutable  
data class CollectibleItemUiState(  
    val id: String = "",    val title: String = "",    val price: String? = "$100.00",    val aspects: List<Pair<UiText, UiText>> = emptyList(),    val photoUrls: List<String> = emptyList(),    val currentPhotoIndex: Int = 0,    val bookmarkState: BookmarkUiState = BookmarkUiState(),)  
```  
  
## Domain layer  
  
The domain layer is responsible for _middleware_: encapsulated business logic that can be shared by view models. The domain layer is often considered to be "optional", but in an app of this size, it's a necessity. The classes that make up the domain layer go by many names ("helper", "util", "interactor", "use case") but ultimately we write domain layer classes for these benefits:  
  
- Avoiding duplication  
- Better readability in VMs and other classes that depend on the encapsulated logic  
- Improved testability, since domain classes can be stubbed/mocked in tests  
- Better separation of responsibility  
  
Follow this simple domain layer rule: If you are about to duplicate code, **don't**. Encapsulate the logic into a use case and provide it as a dependency where its needed.  
  
### A note on "use cases"  
  
In collectibles projects using this architecture, "_verb in present tense + noun/what (optional) + UseCase_" is a naming convention we use to name domain layer classes. That said, we do not use the Dr. Bob clean code approach to writing use cases, mainly because we've found that it increases complexity with limited benefits. So for our purposes, a "use case" is simply a class of encapsulated logic.  
  
## Which layer holds View Models?  
  
Great question! This is something that has been debated among Android developers for centuries. There are multiple ways to think of the view models placement, all with valid arguments. For example:  
  
- The [Android Devs Architecture Guide](https://developer.android.com/topic/architecture#modern-app-architecture:~:text=State%20holders%20(such%20as%20ViewModel%20classes)%20that%20hold%20data%2C%20expose%20it%20to%20the%20UI%2C%20and%20handle%20logic.) says VMs are state holders, part of the UI layer, but the AD documentation regularly contradicts this assertion by showing the business logic in the VM, a common practice which is generally encouraged.  
- Since VMs hold business logic and manipulate data, some prefer to think of it as the "edge" of the data layer.  
- The VM is considered by some to be a membrane between the UI layer and the lower layers (domain and data). By this thinking, it doesn't really sit in a layer at all...  
  
There is objectively one answer to this question: **It's not that big a deal**. Deciding how to conceptualize the placement of the VM is less important than properly using the VM. Here's what VMs should do:  
  
- Act as the membrane between UI and data  
- Manage UI state  
- Utilize domain layer components to aid in the above  
  
## Useful Links and References  
  
- Android Developers Architecture Guide - https://developer.android.com/topic/architecture  
- State hoisting - https://developer.android.com/develop/ui/compose/state#state-hoisting  
- Where to hoist state - https://developer.android.com/develop/ui/compose/state-hoisting  
- Business models - https://developer.android.com/topic/architecture/data-layer#business-models  
  
---  
  
# Collectibles Architecture Overview - Unidirectional Data Flow  
  
## Table of Contents  
- [Introduction to UDF](#introduction-to-udf)  
- [Applying UDF with Model View Intent](#applying-udf-with-model-view-intent)  
- [Implementing UDF in code with CollectiblesScaffold](#implementing-udf-in-code-with-collectiblesscaffold)  
- [Best Practices](#best-practices)  
- [Initialization Approaches](#initialization-approaches)  
- [Self-Contained Components](#self-contained-components)  
- [Helpful Links and References](#helpful-links-and-references-1)  
  
## Introduction to UDF  
  
One of the greatest benefits of Compose is that it's **reactive, state-driven** UI paradigm. This means we can manage the state of a compose screen with **Unidirectional Data Flow (UDF)**.  
  
The defining aspect of UDF is that **data always flows in one direction** as it moves through the layers of the architecture. This one-way flow ensures clean, testable, interpretable code that is less prone to error. The Android Developers Architecture Guide offers a great diagram depicting a UDF flow, with the example of a user bookmarking an article:  
  
<img src="https://developer.android.com/static/topic/libraries/architecture/images/mad-arch-ui-udf-in-action.png" alt="UDF in Action" width="400">  
  
Starting with #1 on the left, follow numbers and notice the direction of the arrows. If properly implemented, a UDF pattern always follows this flow. We **specifically engineer** our code to maintain this flow. What are the greater benefits of applying UDF?  
  
- Data consistency - the data layer is single source of truth  
- Testability - State is isolated, and events are clearly defined, making testing easier  
- Maintainability - Once understood and properly implemented, UDF flows are easy to trace, making modification and debugging easier.  
  
## Applying UDF with Model View Intent  
  
There have been many approaches to implementing the UDF pattern (flux, redux, VIPER, RIBs, and yes these are real acronyms), but collectibles relies on an implementation approach called **Model View Intent**. The are many articles on the topic ([here's a good one](https://proandroiddev.com/mvi-architecture-with-kotlin-flows-and-channels-d36820b2028d)), and there's not singular approach to implementing it, but the approach we're taking is one that has proven reliable elsewhere in the ebay app. The name MVI is not very descriptive or useful, so it will not be used in this document beyond this brief explanation.  
  
We can use the same diagram above to describe how MVI works. Each number in this list matches the number in the diagram:  
  
1. There's a UI backed by a `ViewState` (Ui state in the diagram).  
2. The user taps a button. This triggers the UI to send an **event** to the ViewModel. With MVI we used sealed interface classes to represent events. For example, tapping the bookmark button would call `viewModel.onEvent(ToggleBookmark(true))`.  
3. An event is essentially a request from the UI. Upon receiving the event, the view model gets to work performing the necessary work to fulfill this "request". This likely involves interaction with the data layer.  
4. This part is the same. The data layer does its thing.  
5. Again, same deal: The data layer emits updated data to the view model  
6. At this point the View Model communicates back to the UI, completing the loop. However, with MVI there are **two** ways this can occur:  
   7. The view model updates the `ViewState` which triggers recomposition (the preferred and most common way)  
   8. The view model emits a **side effect** (which we call a `viewEffect`). The UI is set up to observe a flow of side effects, and responds with the appropriate reaction when one is received. Similarly to events, effects are represented with sealed interface classes so that when the view receives an effect it can act accordingly.  
  
## Updating State vs Emitting Side Effects  
  
In general the best practice for updating UI is to **update the view state** that the UI relies on. The Android developer documentation insists that this is the best approach because it guarantees that the UI receives updates. Side effects are actually considered an anti-pattern. **However**, relying purely on state updates presents its own troubles, and while an explanation of those troubles is beyond the scope of this guide, they're significant enough that the use of side effects is warranted. There's a great YouTube video discussing this, which will be linked in the references.  
  
**Side Effects** are exactly what they sound like: They are one-off reactions triggered in response to a view model handling an event. Examples of side effects include:  
  
- Showing a snackbar or (god forbid) a toast message  
- Navigating to a screen  
- Opening a dialog or bottomsheet  
- Launching an activity or fragment  
- That's about it.  
  
For pretty much anything else, **updating state is the best, safest approach**.  
  
## Implementing UDF in code with CollectiblesScaffold (or your domain's equivalent)  
  
> Without structure, architecture is just lines on paper  
  
When writing code with this approach, there are a lot of things that, if not correctly applied, can negatively impact your code and, on some occasions, your mental health. To help us apply the pattern with consistency and resilience, we have `CollectiblesScaffold`. This is a **Kotlin delegate** that can be easily applied to a view model to provide the necessary functionality for implementing the architectural pattern described herein.  
  
Below are examples of setting up and using CollectiblesScaffold. Please note that this is a series of examples. Please don't put all of your UDF code in a single file.  
  
```kotlin  
// Before you apply the delegate, you need a few things: First, make sure you have a view state class  
// for the screen.  
  
@Immutable  
data class CollectibleItemViewState(  
    val uiState: UiState,    val collectibleItemUiState: CollectibleItemUiState,    val isBookmarked: Boolean = false,    val currentMenu: CollectionsOptionsMenuState? = null)  
  
// Next, define a sealed interface for your events:  
  
sealed interface CollectibleItemDetailsEvent {  
    data class UpdateCurrentImageIndex(val index: Int) : CollectibleItemDetailsEvent    data class UpdateActionBar(val title: String?, val isUpAsClose: Boolean) : CollectibleItemDetailsEvent    data class ToggleBookmark(val isBookmarked: Boolean) : CollectibleItemDetailsEvent}  
  
// Finally, define another sealed interface for effects:  
  
sealed interface CollectibleItemDetailsEffect {  
    data class ShowSnackBar(val message: UiText) : CollectibleItemDetailsEffect}  
  
  
// Step 1: Implement the CollectiblesScaffold interface and apply the collectiblesScaffold() delegate with an initial view state.  
// Your best bet is to copy this from here, or elsewhere in the code.  
//  
// In the generics brackets of CollectiblesScaffold<>, include the following, in this order:  
//     1. The view state class being used for the screen's state  
//     2. The event interface name  
//     3. The effects interface name  
// // in the collectiblesScaffold() function (after the "by" keyword), include an initial state for your state flow.  
  
class CollectibleItemViewModel @Inject constructor(  
    private val collectibleItemRepository: CollectibleItemRepository,    // ...) : ViewModel(),  
    CollectiblesScaffold<CollectibleItemViewState, CollectibleItemDetailsEvent, CollectibleItemDetailsEffect> by collectiblesScaffold(        initialState = CollectibleItemViewState(            uiState = UiState.INITIAL,            collectibleItemUiState = CollectibleItemUiState(),            currentMenu = null,        )    ) {  
    // Step 2: Override onEvent and provide a when statement for handling each event.  
    override fun onEvent(event: CollectibleItemDetailsEvent) {        when (event) {            is CollectibleItemDetailsEvent.UpdateCurrentImageIndex -> updateCurrentImageIndex(event.index)            is CollectibleItemDetailsEvent.UpdateActionBar -> updateActionBar(event.title, event.isUpAsClose)            is CollectibleItemDetailsEvent.ToggleBookmark -> toggleBookmark(event.isBookmarked)        }    }  
    // Suppose the user clicked a bookmark, which triggers onEvent(ToggleBookmark(true)). This method would be called.    // If successful, we update the view state, which triggers a recompose. See `updateBookmarkState()` for how this works.    // If it fails, we call showSnackbar(). Take a look at that method's comment to see how to emit a view effect.  
    private fun toggleBookmark(enabled: Boolean) {        viewModelScope.launch {            bookmarkingUseCase.toggleBookmark(viewState.value.collectibleItemUiState.id, enabled)                .collect { bookmarkingResult ->                    when (bookmarkingResult) {                        is BookmarkingResult.Success -> updateBookmarkState(true)                        is BookmarkingResult.Error -> showSnackBar(UiText.StringResource(R.string.dc_folders_general_error))                    }                }        }  
    }  
    // To update the state, we use updateViewState {}. Under the hood this is calling update {}    // on the viewState's StateFlow, so it works just the same way. Updating the underlying state flow    // will trigger a recompose on the screen utilizing the View State.  
    private fun updateBookmarkState(isBookmarked: Boolean) {        updateViewState {            it.copy(bookmarkingEnabled = isBookmarked)        }    }  
    // We can show a snackbar by emitting a view effect, as shown within this method.  
    private fun showSnackBar(message: UiText) {        viewModelScope.emitViewEffect(CollectibleItemDetailsEffect.ShowSnackBar(message))    }}  
  
// Here is the composable that consumes the viewState provided by the view model above. We've uses state hoisting  
// to ensure the view model is not directly referenced here, but assume that all of this composable's parameters  
// are provided by the view model from elsewhere.  
//  
// "viewState" is the same view state we updated in the VM. When the viewState updates, this screen recomposes.  
// "viewEffects" is also from the VM. Both of these are provided by the collectiblesScaffold() delegate. See the comment  
// above "ObserveViewEffect" in the code below to see how we observe and handle effects.  
  
@Composable  
fun CollectibleItemDetailsScreen(  
    viewState: CollectibleItemViewState,    viewEffects: Flow<CollectibleItemDetailsEffect>,    onPageChange: (Int) -> Unit = {},    onImageClick: (Int) -> Unit = {},    onBookmarkToggle: (Boolean) -> Unit = {}) {  
    val context = LocalContext.current    val coroutineScope = rememberCoroutineScope()    val bottomSheetState = rememberModalBottomSheetState(initialValue = ModalBottomSheetValue.Hidden)    val scaffoldState = rememberScaffoldState()  
    // To react to effects, collect the sideEffects flow in a LaunchedEffect as shown below:  
    LaunchedEffect(sideEffects) {        sideEffects.collect { effect ->            when (effect) {                is CollectibleItemDetailsEffect.ShowSnackBar -> {                    scaffoldState.snackbarHostState.showSnackbar(                        message = effect.message.asString(context),                        duration = SnackbarDuration.Short,                    )                }            }        }    }}  
  
// You may have noticed in the code above that the "effects" being addressed by the when  
// statement are `CollectibleLibraryEvent` objects. There are some instances where we may  
// "passthrough events", where an event passes through the VM and is emitted as an effect.  
// This is not common, and usually applies to situations where a compose screen needs to // trigger actions in a fragment or activity which require dependencies at that level.  
// For example, "OpenFolder" uses intentProvider, which we don't want to pass directly  
// to the view model, so using the UDF flow is appropriate here.  
//  
// To allow for an item to be an both an event and an effect, leave an interface unsealed:  
// CollectibleLibraryEffect is an unsealed interface that can be applied to any "passthrough"  
// event.  
  
interface CollectibleLibraryEffect : CollectibleLibraryEvent  
  
  
sealed interface CollectibleLibraryEvent {  
  
    data class BuildListItems(val folders: List<CollectibleFolder>) : CollectibleLibraryEvent  
    // OpenFolder extends CollectibleLibraryEffect, which itself extends `CollectibleLibraryEvent,    // thereby making OpenFolder a "passthrough" event.    data class OpenFolder(val folderId: String) : CollectibleLibraryEffect}  
```  
  
## Best Practices  
  
- UDF works best when applied at the screen level. Use it when you are implementing something that involves a screen-level UI with its own view state and view model.  
- Once you've applied UDF to a screen and view model, **USE IT AS MUCH AS POSSIBLE**. There is no event too small to benefit from UDF. It is important to use it whenever you can because:  
  - It keeps the code consistent, clean, and easy to interpret  
  - It makes testing easier, because the consistency of the flow pattern also means testing patterns are consistent  
  - It makes reviewing PRs easier and more effective, because when everyone understands the flow, the architecture itself becomes "invisible", and the unique properties of the PR become more pronounced.  
- We want to apply the architecture as much as possible, _within reason_. Here are a few situations where you should not use the full UDF flow:  
  - Don't use the flow to communicate within the UI layer unless it is necessary to do so. For example, if clicking button is supposed to display a help text dialog, and both the button and the dialog do not require any content from the view model, there's no need to emit an event for this. In simpler terms: **If you emit an event that does nothing other than triggering an effect, you probably don't need the event**.  
  - Some exceptions to the above rule: Launching activities and fragments often require the use of ebay's UI-level dependencies, such as `IntentProvider`.  
- **Use effects sparingly**. Remember that the recommended practice for reacting to events is to **update the view state**. Exhaust every view state option you have before resorting to effects.  
- **Avoid event / effect chains**. This is when an event triggers an effect which triggers a different event which triggers a different effect. The shortest distant between two points is a line. If you are chaining your effects, consider the following:  
  - Would it make more sense to update the view state? For example, progress indicators can be enabled/disabled during loading using a view state value. No effect is needed.  
  - Are you using an effect chain to trigger multiple processes within a view model? If so, skip the chaining and just do the work in the view model.  
  
## Initialization Approaches  
  
Modern ViewModels in our architecture use specific patterns for initializing content and managing reactive data flows. These patterns provide clean separation of concerns and enable powerful reactive programming capabilities.  
  
### InitContent Events with Refresh Triggers  
  
The standard approach for initializing ViewModel content uses a combination of `InitContent` events and refresh triggers. This pattern provides clean separation between initialization and data fetching, while enabling easy refresh capabilities.  
  
**The Refresh Function**  
  
Every ViewModel that needs to fetch data should have a `refresh()` function:  
  
```kotlin  
private val refreshTrigger = MutableSharedFlow<Unit>()  
private fun refresh() {  
    viewModelScope.launch {  
        refreshTrigger.emit(Unit)    }  
}  
```  
  
**Data Flow Observing Refresh Trigger**  
  
Create flows that observe the `refreshTrigger` and fetch data when it emits:  
  
```kotlin  
private val dataUpdates = refreshTrigger.flatMapLatest {  
    // When refreshTrigger emits, this block executes    dataRepository.fetchData()}.onEach { data ->  
    // Update view state with new data    updateViewState { it.copy(data = data) }  
}.launchIn(viewModelScope)  
```  
  
**InitContent Event Pattern**  
  
Handle initialization through a dedicated event:  
  
```kotlin  
override fun handleEvent(event: MyScreenEvent) {  
    when (event) {        is MyScreenEvent.InitContent -> initContent(event.input)  
        // ... other events    }}  
  
private fun initContent(input: MyScreenInput) {  
    // Store any initialization data    _input.value = input        // Trigger initial data fetch  
    refresh()}  
```  
  
**Key Benefits:**  
- Clean separation between initialization and data fetching  
- Easy refresh capability for pull-to-refresh or retry scenarios  
- Reactive data loading that responds to triggers  
- Testable initialization flow  
  
### Reactive Updates  
  
Use reactive flows to automatically trigger refreshes when certain conditions change. When `refresh()` is called, it emits to `refreshTrigger`, which causes any flows observing `refreshTrigger` to "refresh" (re-execute their `flatMapLatest` block), leading to data being refetched and the view state being updated.  
  
**Filter Updates Pattern**  
  
Observe view state changes and automatically trigger refreshes:  
  
```kotlin  
private val filterUpdates = viewState.mapNotNull { it.filterState }  
    .distinctUntilChanged()    .drop(1) // Skip initial emission to avoid triggering on initialization    .onEach { refresh() } // This calls refresh(), which triggers refreshTrigger    .launchIn(viewModelScope)```  
  
**Complex Reactive Flows**  
  
For more complex scenarios, create multiple reactive flows that respond to different triggers:  
  
```kotlin  
// Observe grader selection changes and update grade options  
private val graderUpdates = viewState.mapNotNull { it.gradeFilterState?.selectedGraderId }  
    .distinctUntilChanged()    .flatMapLatest { graderId ->        dataProvider.graderGrades(graderId)    }    .onEach { gradeData ->        updateViewState { it.copy(gradeOptions = gradeData) }    }    .launchIn(viewModelScope)  
```  
  
**Key Concepts:**  
- Use `distinctUntilChanged()` to avoid unnecessary triggers  
- Use `drop(1)` when you want to skip the initial emission  
- Chain reactive flows to create complex data dependencies  
- Each flow should have a single responsibility  
  
## Self-Contained Components  
  
Self-contained components are reusable UI components that manage their own state and follow the same UDF pattern as screens. They use a factory pattern that parallels the navigation architecture used for screens, providing clean separation of concerns and enabling easy integration across different parts of the app.  
  
### Component Factory Interface  
  
The factory interface defines the component's public API and serves as the contract for how other parts of the app interact with the component:  
  
```kotlin  
/**  
 * Factory for creating condition selection components */interface ConditionSelectionFactory {  
        /**  
     * Condition Selection dropdown field component.     * This is the entry point that shows the current selection and opens the flow when clicked.     */    @Composable    fun HostedComponent(        modifier: Modifier,        viewModelStoreOwner: ViewModelStoreOwner,        input: ConditionSelectionInput,        showEntryPointField: Boolean,        isSheetVisible: Boolean,        onDismiss: () -> Unit,        onSelectionChanged: (selection: ItemConditionModel) -> Unit,    )}  
  
/**  
 * Input data for Condition Selection component. */@Immutable  
data class ConditionSelectionInput(  
    val categoryId: String,    val itemConditionModel: ItemConditionModel?,)  
```  
  
**Key Design Principles:**  
- **Clear Input Contract**: `ConditionSelectionInput` defines exactly what data the component needs  
- **Callback Communication**: Components communicate back to parents through callback functions  
- **Lifecycle Management**: `ViewModelStoreOwner` allows proper ViewModel scoping  
  
### Factory Implementation  
  
The factory implementation handles all the component setup and wiring, similar to how navigation files handle screen setup:  
  
```kotlin  
class ConditionSelectionFactoryImpl @Inject constructor(  
    private val conditionSelectionViewModelFactory: ConditionSelectionViewModel.Factory,) : ConditionSelectionFactory {  
  
    @Composable    override fun HostedComponent(        modifier: Modifier,        viewModelStoreOwner: ViewModelStoreOwner,        input: ConditionSelectionInput,        showEntryPointField: Boolean,        isSheetVisible: Boolean,        onDismiss: () -> Unit,        onSelectionChanged: (selection: ItemConditionModel) -> Unit,    ) {        // Component setup - similar to navigation file setup        val navController = rememberNavController()        val conditionSelectionInput by remember(input) { mutableStateOf(input) }  
        // ViewModel creation with proper scoping        val viewModel = viewModel<ConditionSelectionViewModel>(            viewModelStoreOwner = viewModelStoreOwner,            factory = ConditionSelectionViewModel.createFactory(                assistedFactory = conditionSelectionViewModelFactory,            ),        )                val viewState by viewModel.viewState.collectAsStateWithLifecycle()  
        val sheetState = rememberReactiveSheetState()        var sheetVisible by remember(isSheetVisible) { mutableStateOf(isSheetVisible) }  
        // Side effect handling - component's internal navigation and external communication        LaunchedEffect(viewModel.sideEffects) {            viewModel.sideEffects.collect { sideEffect ->                when (sideEffect) {                    is ConditionSelectionSideEffect.ConfirmSelection -> {                        onSelectionChanged(sideEffect.itemCondition)                        onDismiss()                        sheetVisible = false                    }                    is ConditionSelectionSideEffect.CloseFlow -> {                        onDismiss()                        sheetVisible = false                    }                    // Internal navigation within component                    is ConditionSelectionSideEffect.NavigateToConditionType -> {                        navController.navigateToConditionTypeScreen()                    }                    is ConditionSelectionSideEffect.NavigateToGraderSelection -> {                        navController.navigateToGraderSelectionScreen()                    }                    // ... other internal navigation                }            }        }  
        // Component initialization using InitContent pattern        LaunchedEffect(conditionSelectionInput) {            viewModel.handleEvent(ConditionSelectionEvent.InitContent(conditionSelectionInput))        }  
        // Component UI rendering        if (showEntryPointField) {            ConditionFieldContent(                modifier = modifier,                fieldLabel = stringResource(fieldLabelResId),                conditionText = viewState.conditionFieldDisplayText.asString(),                onClick = { sheetVisible = true }            )        }  
        if (sheetVisible) {            ConditionSelectionSheet(                navController = navController,                sheetState = sheetState,                viewState = viewState,                handleEvent = viewModel::handleEvent,                onDismissSheet = {                    viewModel.handleEvent(ConditionSelectionEvent.CloseConditionSelectionSheet)                }            )        }    }}  
```  
  
**Factory Implementation Responsibilities:**  
- **ViewModel Management**: Creates and scopes the component's ViewModel  
- **Navigation Setup**: Manages internal navigation within the component  
- **State Coordination**: Handles component state and external visibility control  
- **Communication Bridge**: Translates internal side effects to external callbacks  
- **Lifecycle Management**: Ensures proper initialization and cleanup  
  
### Preview Factory  
  
For design system previews and testing, provide a simplified preview implementation:  
  
```kotlin  
@Generated  
class PreviewConditionSelectionFactory: ConditionSelectionFactory {  
    @Composable  
    override fun HostedComponent(  
        modifier: Modifier,        viewModelStoreOwner: ViewModelStoreOwner,        input: ConditionSelectionInput,        showEntryPointField: Boolean,        isSheetVisible: Boolean,        onDismiss: () -> Unit,        onSelectionChanged: (ItemConditionModel) -> Unit,    ) {        if (showEntryPointField) {            ConditionFieldContent(                modifier = modifier,                fieldLabel = "Condition",                conditionText = "Ungraded",                onClick = {},  
            )        }    }}  
```  
  
**Preview Factory Benefits:**  
- **Fast Previews**: No ViewModel or complex state management  
- **Static Content**: Shows the component in a known state  
- **Design Validation**: Allows designers to see component appearance  
- **Testing Support**: Can be used in screenshot tests  
  
### Component Integration  
  
Parent screens integrate components through dependency injection and callback handling:  
  
```kotlin  
class ParentScreen @Inject constructor(  
    private val viewState: ParentScreenViewState,    private val conditionSelectionFactory: ConditionSelectionFactory,    private val viewModelStoreOwner: ViewModelStoreOwner, // usually the current screen's backstack entry    //...) {  
        @Composable  
    fun ParentScreenContent() {        var selectedCondition by remember { mutableStateOf<ItemConditionModel?>(null) }        var showConditionSelection by remember { mutableStateOf(false) }                // Use the component through its factory  
        conditionSelectionFactory.HostedComponent(            modifier = Modifier.fillMaxWidth(),            viewModelStoreOwner: ViewModelStoreOwner,            input = viewState.conditionSelectionInput, // As a view state property, the input can be managed by the view model            showEntryPointField = true,            isSheetVisible = showConditionSelection,            onDismiss = { showConditionSelection = false },            onSelectionChanged = { condition ->                selectedCondition = condition                // Handle the selection in parent screen                handleConditionSelected(condition)            }        )    }}  
```  
  
### Architecture Parallels with Screens  
  
The self-contained component architecture directly parallels screen architecture:  
  
| **Screen Architecture** | **Component Architecture** | **Purpose** |  
|------------------------|----------------------------|-------------|  
| Route definition (`@Serializable data class`) | Factory interface | Defines the contract and input parameters |  
| Navigation file (`NavGraphBuilder.screenName()`) | Factory implementation | Handles setup, wiring, and lifecycle management |  
| Preview composables | Preview factory | Provides static implementations for design/testing |  
| Navigation parameters | Component callbacks | Enables communication with parent |  
| Screen ViewModel | Component ViewModel | Manages state using the same UDF patterns |  
  
**Key Insight**: The factory implementation serves the same role for components that navigation files serve for screens - they handle the setup, wiring, and lifecycle management, keeping the complexity hidden from the consuming code.  
  
### Component Benefits  
  
- **Reusability**: Can be used across different screens with consistent behavior  
- **Isolation**: Component state is completely separate from parent screen state    
- **Testability**: Component logic can be tested independently through the factory interface  
- **Clear Contracts**: Factory interface and callbacks define clear communication boundaries  
- **Maintainability**: Changes to component internals don't affect consuming screens  
- **Consistency**: Follows the same architectural patterns as screens, making it familiar to developers  
  
### Other Examples in the Codebase  
  
- **UnifiedPriceGuidanceFactory**: Provides price guidance components with market data, recent sales, and filtering capabilities  
- **ShowGradedCardFactory**: Handles the display and interaction logic for graded collectible cards  
  
## Helpful Links and References  
  
- Android Developers Architecture Guide - UI Layer - https://developer.android.com/topic/architecture/ui-layer  
- MVI Architecture with Kotlin Flows and Channels - https://proandroiddev.com/mvi-architecture-with-kotlin-flows-and-channels-d36820b2028d  
- [Are one-time events an anti-pattern?](https://youtu.be/njchj9d_Lf8?si=szeJCvB3aw1uY7yS)  
  
---  
  
# Collectibles Testing Best Practices  
  
## Table of Contents  
- [General Unit Testing Guidelines](#general-unit-testing-guidelines)  
- [UI Testing With the Page Object Pattern](#ui-testing-with-the-page-object-pattern)  
  
## General Unit Testing Guidelines  
  
- Prefer fakes over mocks  
- Prefer mockk over mockito  
- Use turbine to test flows  
- Prefer UI unit tests with robolectric over instrumented UI tests  
- Reduce redundant code by writing dummy data objects that can be used across multiple tests  
- Reduce redundant code by writing reusable fakes rather than creating unique instances for each test class, when possible  
- When possible, use fixtures to reduce the need for dummy data objects  
  
## UI Testing With the Page Object Pattern  
  
Page Object is a design pattern commonly used in automation testing that offers the following benefits:  
  
- Reduces duplicated code in tests  
- Makes tests easier to read  
- Makes tests easier to fix: when UI changes, only the page objects need to be changed  
- Clean separation between UI matchers (what the page elements _are_) and test assertions (what the page elements _do_)  
  
### Defining Page Objects Models  
  
Page objects Models are simple classes containing getter methods for elements on a page. There are a few general rules:  
  
- Page objects Models (POMs) generally represent _pages_ (screens), so a best practice is to create a page object for each screen. **However...**  
- For better reuse, it sometimes makes sense to create POMs for **components** of a page. For example, we have a page object for the Action Bar, since most screens rely on it. This way we don't make duplicate action bars in each POM.  
- Tests can (and often do) rely on multiple POMs. Always prefer reuse over writing new stuff.  
- POMs can mix compose and espresso matchers. Investigate the code for examples of both usages  
- **Avoid assertions in POMs**. Assertions should happen in the test, since that's the point of testing. POMs elements should rely on matchers  
  
The following is an example of a Page Object Model:  
  
```kotlin  
class CollectibleLibraryPageObject(private val composeTestRule: ComposeTestRule) {  
  
    /* A composable element matcher. These usually return a SemanticsNodeInteraction. */    fun folderMaxNotice(): SemanticsNodeInteraction = composeTestRule        .onNodeWithText("Folder limit reached. Delete a folder to create a new one.", useUnmergedTree = true)  
   /* Sometimes its necessary for an element matcher to take in a parameter. This allows for better reusability. */    fun folderListItemCount(count: Int): SemanticsNodeInteraction = composeTestRule        .onNodeWithText("($count)", useUnmergedTree = true)  
    /* An espresso element matcher. These usually return a Matcher<View>. */    fun generalErrorSnackBarMessage(): Matcher<View> = allOf(        ViewMatchers.withId(com.google.android.material.R.id.snackbar_text),        ViewMatchers.withText("Changes could not be saved, please try again.")    )}  
```  
  
### Using POMs in tests  
  
Using POMs in tests is simple:  
  
- Add an instance of any POM you need to your test class  
- Use the POMs element matchers, performing your assertions on them.  
- Remember again that tests can (and often do) use multiple POMs.  
  
Here's an example of the POM from earlier being used in a test. This is code that was modified for the sake of example, so some non-relevant code is not included:  
  
```kotlin  
class CollectibleLibraryUiTest {  
        @get:Rule  
    val rule = RuleChain.outerRule(InstrumentationBaseRule())        .around(injectionInitializerRule)  
    @get: Rule    val mockitoTestRule: MockitoTestRule = MockitoJUnit.testRule(this)  
    @get:Rule    val composeTestRule = createEmptyComposeRule()          
    private val createRenameBottomSheetPageObject = CreateRenameFolderBottomSheetContentPageObject(composeTestRule)  
    private val collectibleLibraryPageObject = CollectibleLibraryPageObject(composeTestRule)  
    @Test    fun generalErrorSnackBarMessageDisplayedOnCreateFolderError() {        launchCollectionActivity()  
        /**         * Page object element matchers can be asserted on, and they can be used for navigation. Here are using elements         * from the CreateRenameBottomSheetPageObject to enter text into a text field and click a button.          */        createRenameBottomSheetPageObject.run {  
            textField()                .performClick()                .performTextReplacement("My folder")  
            saveButton().performClick()  
        }  
        /**         * Once the button is clicked, we use a matcher from the CollectibleLibraryPageObject to check if the general         * error snackbar message is displayed. Note that this uses espresso.         */        onView(collectibleLibraryPageObject.generalErrorSnackBarMessage()).checkDisplayed()    }}  
```  
  
### Accessing the SemanticsMatcher of a POM element  
  
Occasionally there are compose testing functions that require a `SemanticsMatcher`, but generally our POM element getters return `SemanticsNodeInteraction`. To help address this issue without duplicating matcher code, you can use `PageElement`, a convenience class that offers access to the matcher and the interaction. Here's how to use it:  
  
```kotlin  
// Page Object  
class CollectibleItemDetailsPageObject(val testRule: ComposeTestRule) {  
  
    /* Instead of returning a SemanticsNodeInteraction, we create and return a PageElement, like so */    fun psaMigrationDetailsBannerTitle(title: String): PageElement {        return hasText(title).toPageElement(testRule, useUnmergedTree = true)    }    // ...    }  
  
// The Test  
@Test  
    fun testPsaMigrationDetails() {        composeTestRule.setContent {            PreviewCollectibleItemDetailsScreen()        }  
        composeTestRule.run {            pageObject.run {                /* performScrollToNode requires a SemanticsMatcher. We can provide one using the .matcher property of the page element. */                getReadyStateColumn().performScrollToNode(getPsaMigrationDetailsButton("Go to PSA").matcher)  
                /* When we want the element's node interaction, we can use the .nodeInteraction property */                psaMigrationDetailsBannerTitle("PSA Notice Title").nodeInteraction.assertExists()  
                /* You can also get the nodeInteraction by "invoking" the PageElement instance. Call .invoke(), or add parenthesis (shown here) */                val psaMigrationDetailsButton = getPsaMigrationDetailsButton("Go to PSA")                psaMigrationDetailsButton()().assertExists()            }        }    }  
```  
  
---  
  
# Collectibles UI Layer Architecture  
  
## Table of Contents  
- [Naming Events and Effects](#naming-events-and-effects)  
- [Navigation](#navigation)  
- [Activity-Level State](#activity-level-state)  
  
## Naming Events and Effects  
  
Events and effects should be named in a way that makes them self-documenting. If done correctly, it is easy for engineers to read code and determine exactly what's going on. Follow these rules when naming events and effects:  
  
| DO ✅ | DON'T ⛔ |  
|-------|---------|  
| **DO**: Name **events** according to what is being **requested**.<br/>`data object AddToFolder: CollectibleItemDetailsEvent` | **DON'T**: Name events according to the UI interaction that calls them.<br/>`data object AddToFolderButtonClick: CollectibleItemDetailsEvent` |  
| **DO:** Name **effects** according to the **response** (the expected outcome)<br/>`data class ShowSnackBar(val message: UiText)` | **DON'T**: Name effects according to what happened to trigger them:<br/>`data class OnItemAddedToFolder(val message: UiText)` |  
| **DO:** Keep event and effect names clear and concise, avoiding redundant naming. | **DON'T**: Use the words "Event" and "Effect" in the names. |  
  
### Additional Tips  
  
- Generally you don't need to add kdocs to an event or effect, but add them if they help describe the purpose of the parameters provided in the event/effect.  
- If a screen has a lot of events and effects, organize them alphabetically.  
  
## Navigation  
  
### CollectiblesNavHost  
  
`CollectibleActivity` relies on a single Navigation Graph: `CollectiblesNavHost` (we'll call this the "Nav Host" herein). Here's what to know:  
  
- Any compose screen we want to display in this activity must provide a **destination** to the Nav Host.  
- The Nav Host is the bridge between the Activity and the composable screens. The state, event handlers, effect flows, and other data and lambda functions are set up in the Activity and provided to screens via the Nav Host's parameters.  
  
### Screen Navigation Files  
  
As mentioned before, each screen needs to provide a destination the NavHost. Additionally, it's helpful to offer a convenient way to navigate to each screen. Both of these considerations are handled in a screens **Navigation File**. This is a kotlin file that hosts all the navigation-related logic for a screen. Let's see an example. Be sure to read the comments for explanations.  
  
```kotlin  
/**  
 * We define type-safe routes using @Serializable data classes. This provides compile-time safety * and eliminates the need for manual string manipulation and argument parsing. */@Serializable  
data class AddNotesRoute(  
    val collectibleId: String?,    val note: String? = null,)  
  
/**  
 * This extension function allows us to easily navigate to the screen, providing the route object directly. * The navigation library handles serialization automatically. */fun NavController.navigateToAddNotesScreen(addNotesRoute: AddNotesRoute, navOptions: NavOptions? = null) =  
    navigate(addNotesRoute, navOptions)  
/**  
 * This is the "destination" definition for the screen. All the data and lambdas we need for the screen are passed down * from the NavHost, which gets them from the activity. Note that we use composable<T> with our route type for type safety.  
 */fun NavGraphBuilder.addToNotesScreen(  
    viewModelFactory: ViewModelProvider.Factory,    navHostController: NavHostController,    updateActionBar: (ActionBarState) -> Unit,    showCollectiblesSnackBar: (CollectiblesSnackBarState) -> Unit,) {  
    composable<AddNotesRoute> { backStackEntry ->        /**         * Extract route parameters using toRoute<T>() - this provides type-safe access to our route data         */        val initialData = backStackEntry.toRoute<AddNotesRoute>()  
        /**         * Create ViewModel with proper scoping. Using backStackEntry as viewModelStoreOwner ensures         * the ViewModel is scoped to this specific navigation entry.         */        val viewModel = viewModel<AddNotesViewModel>(            viewModelStoreOwner = backStackEntry,            factory = viewModelFactory,            key = initialData.collectibleId        )  
        val viewState = viewModel.viewState.collectAsStateWithLifecycle().value  
        /**         * Initialize content using the InitContent event pattern with the extracted route data         */        LaunchedEffect(Unit) {            viewModel.handleEvent(AddNotesEvent.InitContent(initialData))        }  
        /**         * Update action bar using rememberUpdatedState to ensure we have the latest callback         */        val updateActionBarCallback by rememberUpdatedState(updateActionBar)        LaunchedEffect(Unit) {            updateActionBarCallback(viewState.actionBarState)        }  
        AddNotesScreen(            state = viewState,            handleEvent = viewModel::handleEvent,            sideEffects = viewModel.sideEffects,            onDismiss = { message ->                showCollectiblesSnackBar(CollectiblesSnackBarState(message = message))                navHostController.apply {                    previousBackStackEntry.withRefresh()                    popBackStack()                }            },        )    }}  
```  
  
### Good Navigation Hygiene  
  
Some important things to keep in mind  
  
- **Don't pass view models into Screen composables**. Doing so couples child composables to the VM, makes them less maintainable, and makes them difficult to test.  
- **Don't pass UDF framework components further than the screen composable**. It's fine to provide the screen's viewState, handleEvent, and sideEffects to the screen composable, but they should not be passed any further. Instead, child composables should apply [state hoisting](https://developer.android.com/develop/ui/compose/state#state-hoisting).  
  
## Activity-Level State  
  
Often we need to update UI that is shared by all screens. This can include:  
  
- Action Bar (title, home/"Up" button style, other Decor options)  
- Overflow menu (what options should be shown)  
- Bottom Nav/Nav Rail (whether or not to display it)  
  
For these situations we have an activity-level view model which handles state changes using the same UDF pattern:  
  
- **CollectibleActivityViewModel** - handles business logic, acts as bridge between CollectibleActivity and the data layer  
- **CollectibleActivityState** - Activity-level view state, holds data used to populate activity-level views  
- **CollectibleActivityEvent, CollectibleActivityEffect** - Event and Effect classes used to request state updates and respond to state changes  
  
### Action Bar State  
  
Updating the action bar follows the UDF pattern like so:  
  
1. The screen in need of an action bar change requests to update the action bar, providing an `ActionBarState` instance  
2. The request is sent as an event to `CollectibleActivityViewModel`  
3. The VM updates its `CollectibleActivityState.actionBarState` with `ActionBarState` instance provided in the event  
4. `CollectibleActivity` observes changes to the VMs `CollectibleActivityState.actionBarState`, and updates the action bar title and options based on the latest content.  
  
#### Action Bar Setup  
  
**Step 1: Make sure your Screen's view state has an actionBarState property with a default value.**  
  
Note: If your screen has static action bar content, use the default value to set the static properties.  
  
```kotlin  
@Immutable  
data class AddNotesUiState(    // properties ...  
    val actionBarState: ActionBarState = ActionBarState(  
        title = UiText.StringResource(R.string.dc_ilv_add_a_note),        showUpAsClose = true,    )  
    // ... more properties)  
```  
  
**Steps 2 and 3: Provide updateActionBar lambda to the screen's NavGraph destination builder, then use a `LaunchedEffect` to call `updateActionBar` with your screen's actionBarState.**  
  
Add `updateActionBar` to your screen's NavGraph destination builder like so:  
  
```kotlin  
fun NavGraphBuilder.addToNotesScreen(  
    navHostController: NavHostController,    addNotesUiState: AddNotesUiState,    sideEffects: Flow<CollectibleItemDetailsEffect>,    updateActionBar: (ActionBarState) -> Unit, // lambda for updating action bar, to be provided by NavHost    onSaveClick: (String) -> Unit,) {  
    composable<AddNotesRoute> {  
       // Calling updateActionBar with the screen state's actionBarState        LaunchedEffect(Unit) { updateActionBar(addNotesUiState.actionBarState) }  
        AddNotesScreen(            state = addNotesUiState,            sideEffects = sideEffects,            onSaveClick = onSaveClick,            onDismiss = { navHostController.popBackStack() },        )    }}  
```  
  
Provide the `CollectibleNavHost`'s `updateActionBar` lambda property to your screen's destination function call:  
  
```kotlin  
@Composable  
fun CollectiblesNavHost(  
    // ...    navHostController: NavHostController,    startDestination: String = TAB_HOST_ROUTE,    updateActionBar: (ActionBarState) -> Unit, // The lambda to pass to your screen    itemDetailsViewState: CollectibleItemViewState,    handleItemDetailsEvent: (CollectibleItemDetailsEvent) -> Unit,    itemDetailsSideEffects: Flow<CollectibleItemDetailsEffect>,    // ...) {  
    NavHost(navController = navHostController, startDestination = startDestination) {  
        // ...        addToNotesScreen(            navHostController = navHostController,            addNotesUiState = itemDetailsViewState.addNotesUiState,            sideEffects = itemDetailsSideEffects,            updateActionBar = updateActionBar, // Pass the lambda to the screen            onSaveClick = { note -> handleItemDetailsEvent(CollectibleItemDetailsEvent.SaveNote(note)) },  
        )    }  
}  
```  
  
**That's it! 👏**  
  
**Set a key in your `LaunchedEffect` if you need your action bar to update in response to state changes**  
  
For example, this LaunchEffect will call `updateActionBar` whenever `viewState.actionBarState` is updated:  
  
```kotlin  
LaunchedEffect(key = viewState.actionBarState) {  
            updateActionBar(viewState.actionBarState)}  
```  
  
#### Additional Considerations  
  
- **ALWAYS include an ActionBarState property for screens**, and be sure to update the action bar state. Otherwise the current action bar settings will be retained when navigating to/from a screen.  
  
### Options Menu State  
  
Note that "Options menu refers to the "Overflow" menu. These terms can be used interchangeably, but herein we will refer to the menu as the "options menu".  
  
Updating the overflow menu follows the UDF pattern like so:  
  
1. The screen in need of an options menu change requests to update the options menu, providing an `OptionsMenuState` instance  
2. The request is sent as an event to `CollectibleActivityViewModel`  
3. The VM updates its `CollectibleActivityState.optionsMenuState` with `OptionsMenuState` instance provided in the event  
4. `CollectibleActivity` observes changes to the VMs `CollectibleActivityState.optionsMenuState`, calling `invalidateOptionsMenu()` whenever it changes  
5. `invalidateOptionsMenu()` re-calls `onCreateOptionsMenu()`. In `onCreateOptionsMenu()` the `CollectibleActivityState.optionsMenuState` is used to populate the menu  
  
#### Options Menu Setup  
  
**Step 1: Create a menu.xml file for your screen, if needed**  
  
- Create a menu.xml only if your screen has a **unique set of menu options**.  
- If your menu is static (the screen will always have the same options), enable visibility on all items  
- If your menu is dynamic, **disable visibility on all items**. This will allow you to selectively enable/disable visibility through state changes  
- **DO NOT INCLUDE A DSA OPTION**. The "Help & Report" DSA option is taken care of by the activity, so you don't need to include it in your menu xml.  
  
```xml  
<?xml version="1.0" encoding="utf-8"?>  
<menu xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:app="http://schemas.android.com/apk/res-auto">    <item       android:id="@+id/dc_menu_option_enable_collection_updates"       android:title="@string/dc_menu_item_enable_collection_updates"       app:showAsAction="never" />    <item       android:id="@+id/dc_menu_option_whats_new"       android:title="@string/dc_menu_item_whats_new_in_collection"       app:showAsAction="never" /></menu>  
```  
  
**Step 2: Make sure your Screen's view state has an optionsMenu property with a default value.**  
  
Note: If your screen has static options menu content, use the default value to set the static properties.  
  
```kotlin  
@Immutable  
data class TabHostState(    // properties ...  
    val optionsMenu: OptionsMenuState = OptionsMenuState(  
        menu = R.menu.dc_tab_layout_menu,        showAllOptions = true    ),  
    // ... more properties)  
```  
  
**Steps 3 and 4: Provide the updateOptionsMenu lambda and the menuOptionEvents flow to the screen's NavGraph destination builder, then use a `LaunchedEffect` to call `updateOptionsMenu` with your screen's optionsMenuState.**  
  
```kotlin  
fun NavGraphBuilder.tabHostScreen(  
    navHostController: NavHostController,    tabHostState: TabHostState,  
    // ... more parameters    // ... even more parameters  
    updateOptionsMenu: (OptionsMenuState) -> Unit, // lambda for updating the options menu, to be provided by NavHost    menuOptionEvents: Flow<Int>, // flow of menu option click events, to be provided by NavHost) {  
    composable<TabHostRoute> {  
       // Call updateOptionsMenu with the state's optionsMenu        LaunchedEffect(Unit) {            updateOptionsMenu(tabHostState.optionsMenu)        }  
       // Provide the menuOptionEvents flow to the screen composable as a parameter        TabHostScreen(            state = tabHostState,            // ... more parameters          menuOptionEvents = menuOptionEvents, // Providing menuOptionEvents here            // ... even more parameters        )    }}  
```  
  
Provide the `CollectibleNavHost`'s `updateOptionsMenu` lambda and menuOptionEvents flow properties to your screen's destination function call:  
  
```kotlin  
@Composable  
fun CollectiblesNavHost(  
    // ...        navHostController: NavHostController,  
    startDestination: String = TAB_HOST_ROUTE,    updateOptionsMenu: (OptionsMenuState) -> Unit, // The lambda to pass to your screen's destination builder    menuOptionEvents: Flow<Int>, // The flow to pass to your screen, via the screen's destination builder        // ...  
) {  
    NavHost(navController = navHostController, startDestination = startDestination) {  
        // ...        tabHostScreen(          // ...            updateOptionsMenu = updateOptionsMenu,            menuOptionEvents = menuOptionEvents,          // ...        )    }  
}  
```  
  
**Step 5: In your screen composable, add the `ObserveOptionsMenuEvents` flow listener function to react to menu option clicks**  
  
```kotlin  
@Composable  
fun TabHostScreen(  
    //...    menuOptionEvents: Flow<Int>, // Passed in from NavGraph destination builder for screen) {  
    // ...    ObserveOptionsMenuEvents(menuOptionEvents) { optionId ->  
        when(optionId) {            R.id.dc_menu_option_enable_collection_updates -> handleTabHostEvent(                TabHostEvent.OpenBottomSheet(EnableCollectionUpdates())            )            R.id.dc_menu_option_whats_new -> {                handleTabHostEvent(                    TabHostEvent.OpenBottomSheet(Splash())                )                handleSplashEvent(                    SplashEvent.TrackSplashOpened                )            }        }    }  
    // ...}  
```  
  
When an option is clicked, the click event is observed here. Simply use a `when` statement to react to the observed menu IDs  
  
**Useful note: Set a key in your `LaunchedEffect` if you need your options menu to update in response to state changes**  
  
For example, this LaunchEffect will call `updateActionBar` whenever `viewState.actionBarState` is updated:  
  
```kotlin  
LaunchedEffect(key = viewState.optionsMenuState) {  
            updateActionBar(viewState.optionsMenuState)}  
```  
  
#### Additional Considerations  
  
- For screens that have no menu options, provide an empty `OptionsMenuState()` in your screen's view state. This takes care of setting an empty menu.  
- If you need to dynamically control the available options for a screen, be sure to set all items in the menu.xml file to be **disabled**. Then, when setting your screen's options menu state you can use `MenuOption.enabled` to enable the relevant options.  
- Menu items defined in your menu.xml should have their own titles, but if you need to set titles programmatically you can do so by setting `MenuOption.label` on menu items in your options menu state. The activity takes care of the rest.