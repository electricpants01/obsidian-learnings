# Navigation Compose

## Conceptos Básicos

### 1. Setup de Navigation

```kotlin
// En build.gradle
dependencies {
    implementation("androidx.navigation:navigation-compose:2.7.6")
}

// NavHost básico
@Composable
fun App() {
    val navController = rememberNavController()
    
    NavHost(
        navController = navController,
        startDestination = "home"
    ) {
        composable("home") {
            HomeScreen(navController)
        }
        composable("profile") {
            ProfileScreen(navController)
        }
        composable("settings") {
            SettingsScreen(navController)
        }
    }
}
```

### 2. Navegación Básica

```kotlin
@Composable
fun HomeScreen(navController: NavController) {
    Column {
        Button(onClick = {
            navController.navigate("profile")
        }) {
            Text("Go to Profile")
        }
        
        Button(onClick = {
            navController.navigate("settings")
        }) {
            Text("Go to Settings")
        }
    }
}

@Composable
fun ProfileScreen(navController: NavController) {
    Column {
        Text("Profile Screen")
        Button(onClick = {
            navController.popBackStack()  // Volver atrás
        }) {
            Text("Back")
        }
    }
}
```

## Navegación con Argumentos

### 1. Argumentos Obligatorios

```kotlin
@Composable
fun AppNavigation() {
    val navController = rememberNavController()
    
    NavHost(navController = navController, startDestination = "home") {
        composable("home") {
            HomeScreen(
                onUserClick = { userId ->
                    navController.navigate("user/$userId")
                }
            )
        }
        
        composable(
            route = "user/{userId}",
            arguments = listOf(
                navArgument("userId") {
                    type = NavType.StringType
                }
            )
        ) { backStackEntry ->
            val userId = backStackEntry.arguments?.getString("userId")
            UserScreen(userId = userId ?: "")
        }
    }
}

@Composable
fun HomeScreen(onUserClick: (String) -> Unit) {
    LazyColumn {
        items(users) { user ->
            UserItem(
                user = user,
                onClick = { onUserClick(user.id) }
            )
        }
    }
}

@Composable
fun UserScreen(userId: String) {
    Text("User ID: $userId")
}
```

### 2. Argumentos Opcionales

```kotlin
NavHost(navController = navController, startDestination = "search") {
    composable(
        route = "search?query={query}",
        arguments = listOf(
            navArgument("query") {
                type = NavType.StringType
                nullable = true
                defaultValue = null
            }
        )
    ) { backStackEntry ->
        val query = backStackEntry.arguments?.getString("query")
        SearchScreen(initialQuery = query)
    }
}

// Navegación
navController.navigate("search?query=kotlin")
navController.navigate("search")  // Sin query
```

### 3. Múltiples Argumentos

```kotlin
composable(
    route = "product/{productId}?color={color}&size={size}",
    arguments = listOf(
        navArgument("productId") {
            type = NavType.StringType
        },
        navArgument("color") {
            type = NavType.StringType
            nullable = true
            defaultValue = "red"
        },
        navArgument("size") {
            type = NavType.StringType
            nullable = true
            defaultValue = "M"
        }
    )
) { backStackEntry ->
    val productId = backStackEntry.arguments?.getString("productId") ?: ""
    val color = backStackEntry.arguments?.getString("color") ?: "red"
    val size = backStackEntry.arguments?.getString("size") ?: "M"
    
    ProductScreen(productId, color, size)
}

// Navegación
navController.navigate("product/123?color=blue&size=L")
```

### 4. Type Safety con Routes (Recomendado)

```kotlin
// Definir rutas como sealed class
sealed class Screen(val route: String) {
    object Home : Screen("home")
    object Profile : Screen("profile")
    data class User(val userId: String) : Screen("user/{userId}") {
        fun createRoute(userId: String) = "user/$userId"
    }
    data class ProductDetail(
        val productId: String,
        val color: String? = null
    ) : Screen("product/{productId}?color={color}") {
        fun createRoute(productId: String, color: String? = null): String {
            val route = "product/$productId"
            return if (color != null) "$route?color=$color" else route
        }
    }
}

// Uso
NavHost(navController = navController, startDestination = Screen.Home.route) {
    composable(Screen.Home.route) {
        HomeScreen(
            onUserClick = { userId ->
                navController.navigate(Screen.User("").createRoute(userId))
            }
        )
    }
    
    composable(
        route = Screen.User("").route,
        arguments = listOf(
            navArgument("userId") { type = NavType.StringType }
        )
    ) { backStackEntry ->
        val userId = backStackEntry.arguments?.getString("userId") ?: ""
        UserScreen(userId)
    }
}
```

## Navegación Avanzada

### 1. popUpTo - Limpiar BackStack

```kotlin
// Ir a login y limpiar todo el backstack
navController.navigate("login") {
    popUpTo(0) { inclusive = true }
}

// Ir a home y limpiar hasta home (inclusive)
navController.navigate("home") {
    popUpTo("home") { inclusive = true }
}

// Evitar múltiples instancias de la misma pantalla
navController.navigate("details") {
    popUpTo("home")
    launchSingleTop = true
}
```

### 2. launchSingleTop

```kotlin
// Solo una instancia de la pantalla en el stack
navController.navigate("profile") {
    launchSingleTop = true
}
```

### 3. saveState y restoreState

```kotlin
// Guardar estado al navegar
navController.navigate("settings") {
    popUpTo("home")
    launchSingleTop = true
    restoreState = true  // Restaurar estado si existe
}

// Al salir, guardar el estado
navController.navigate("home") {
    popUpTo("settings") {
        saveState = true  // Guardar estado al salir
    }
}
```

### 4. Navegación Condicional

```kotlin
@Composable
fun AppNavigation() {
    val navController = rememberNavController()
    val isLoggedIn by viewModel.isLoggedIn.collectAsState()
    
    NavHost(
        navController = navController,
        startDestination = if (isLoggedIn) "home" else "login"
    ) {
        composable("login") {
            LoginScreen(
                onLoginSuccess = {
                    navController.navigate("home") {
                        popUpTo("login") { inclusive = true }
                    }
                }
            )
        }
        
        composable("home") {
            HomeScreen()
        }
    }
}
```

## Deep Links

### 1. Deep Links Implícitos

```kotlin
composable(
    route = "profile/{userId}",
    deepLinks = listOf(
        navDeepLink {
            uriPattern = "myapp://profile/{userId}"
        }
    )
) { backStackEntry ->
    val userId = backStackEntry.arguments?.getString("userId")
    ProfileScreen(userId = userId ?: "")
}

// AndroidManifest.xml
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="myapp" />
    </intent-filter>
</activity>
```

### 2. Deep Links con HTTP/HTTPS

```kotlin
composable(
    route = "article/{articleId}",
    deepLinks = listOf(
        navDeepLink {
            uriPattern = "https://www.myapp.com/article/{articleId}"
        },
        navDeepLink {
            uriPattern = "myapp://article/{articleId}"
        }
    )
) { backStackEntry ->
    val articleId = backStackEntry.arguments?.getString("articleId")
    ArticleScreen(articleId = articleId ?: "")
}

// AndroidManifest.xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data
        android:scheme="https"
        android:host="www.myapp.com" />
</intent-filter>
```

## Nested Navigation (Navegación Anidada)

### 1. Nested Graphs

```kotlin
@Composable
fun AppNavigation() {
    val navController = rememberNavController()
    
    NavHost(
        navController = navController,
        startDestination = "main",
        route = "root"
    ) {
        // Main graph
        navigation(
            startDestination = "home",
            route = "main"
        ) {
            composable("home") {
                HomeScreen(navController)
            }
            composable("profile") {
                ProfileScreen(navController)
            }
        }
        
        // Auth graph
        navigation(
            startDestination = "login",
            route = "auth"
        ) {
            composable("login") {
                LoginScreen(navController)
            }
            composable("signup") {
                SignUpScreen(navController)
            }
        }
        
        // Settings graph
        navigation(
            startDestination = "settings_main",
            route = "settings"
        ) {
            composable("settings_main") {
                SettingsMainScreen(navController)
            }
            composable("settings_account") {
                AccountSettingsScreen(navController)
            }
            composable("settings_privacy") {
                PrivacySettingsScreen(navController)
            }
        }
    }
}

// Navegar a un graph completo
navController.navigate("auth")
navController.navigate("settings")
```

### 2. Bottom Navigation con Nested Graphs

```kotlin
@Composable
fun MainScreen() {
    val navController = rememberNavController()
    
    Scaffold(
        bottomBar = {
            BottomNavigationBar(navController)
        }
    ) { paddingValues ->
        NavHost(
            navController = navController,
            startDestination = "home_graph",
            modifier = Modifier.padding(paddingValues)
        ) {
            // Home graph
            navigation(
                startDestination = "home",
                route = "home_graph"
            ) {
                composable("home") {
                    HomeScreen(navController)
                }
                composable("home_details/{id}") { backStackEntry ->
                    val id = backStackEntry.arguments?.getString("id")
                    HomeDetailsScreen(id = id ?: "", navController)
                }
            }
            
            // Search graph
            navigation(
                startDestination = "search",
                route = "search_graph"
            ) {
                composable("search") {
                    SearchScreen(navController)
                }
                composable("search_results") {
                    SearchResultsScreen(navController)
                }
            }
            
            // Profile graph
            navigation(
                startDestination = "profile",
                route = "profile_graph"
            ) {
                composable("profile") {
                    ProfileScreen(navController)
                }
                composable("edit_profile") {
                    EditProfileScreen(navController)
                }
            }
        }
    }
}

@Composable
fun BottomNavigationBar(navController: NavController) {
    val navBackStackEntry by navController.currentBackStackEntryAsState()
    val currentRoute = navBackStackEntry?.destination?.route
    
    NavigationBar {
        NavigationBarItem(
            selected = currentRoute?.startsWith("home") == true,
            onClick = {
                navController.navigate("home_graph") {
                    popUpTo(navController.graph.findStartDestination().id) {
                        saveState = true
                    }
                    launchSingleTop = true
                    restoreState = true
                }
            },
            icon = { Icon(Icons.Default.Home, contentDescription = null) },
            label = { Text("Home") }
        )
        
        NavigationBarItem(
            selected = currentRoute?.startsWith("search") == true,
            onClick = {
                navController.navigate("search_graph") {
                    popUpTo(navController.graph.findStartDestination().id) {
                        saveState = true
                    }
                    launchSingleTop = true
                    restoreState = true
                }
            },
            icon = { Icon(Icons.Default.Search, contentDescription = null) },
            label = { Text("Search") }
        )
        
        NavigationBarItem(
            selected = currentRoute?.startsWith("profile") == true,
            onClick = {
                navController.navigate("profile_graph") {
                    popUpTo(navController.graph.findStartDestination().id) {
                        saveState = true
                    }
                    launchSingleTop = true
                    restoreState = true
                }
            },
            icon = { Icon(Icons.Default.Person, contentDescription = null) },
            label = { Text("Profile") }
        )
    }
}
```

## Compartir Datos entre Screens

### 1. Usando Saved State Handle

```kotlin
// Screen A - envía datos
navController.currentBackStackEntry
    ?.savedStateHandle
    ?.set("user_data", userData)

navController.popBackStack()

// Screen B - recibe datos
val savedStateHandle = navController.currentBackStackEntry?.savedStateHandle
val userData = savedStateHandle?.get<UserData>("user_data")

// O con StateFlow
val userDataFlow = savedStateHandle
    ?.getStateFlow<UserData?>("user_data", null)
    ?.collectAsState()
```

### 2. Usando Previous BackStack Entry

```kotlin
// Screen A
@Composable
fun ScreenA(navController: NavController) {
    Button(onClick = {
        navController.navigate("screen_b")
    }) {
        Text("Go to B")
    }
    
    // Observar resultado de Screen B
    val result = navController.currentBackStackEntry
        ?.savedStateHandle
        ?.getStateFlow<String?>("result", null)
        ?.collectAsState()
    
    result?.value?.let { data ->
        Text("Result from B: $data")
        // Limpiar después de usar
        navController.currentBackStackEntry
            ?.savedStateHandle
            ?.set("result", null)
    }
}

// Screen B
@Composable
fun ScreenB(navController: NavController) {
    Button(onClick = {
        navController.previousBackStackEntry
            ?.savedStateHandle
            ?.set("result", "Data from B")
        navController.popBackStack()
    }) {
        Text("Send result and go back")
    }
}
```

### 3. Shared ViewModel (ViewModel compartido)

```kotlin
// ViewModel con scope de navigation graph
@Composable
fun MyNavGraph() {
    val navController = rememberNavController()
    
    NavHost(
        navController = navController,
        startDestination = "screen_a",
        route = "my_graph"
    ) {
        composable("screen_a") { backStackEntry ->
            val parentEntry = remember(backStackEntry) {
                navController.getBackStackEntry("my_graph")
            }
            val sharedViewModel: SharedViewModel = hiltViewModel(parentEntry)
            
            ScreenA(navController, sharedViewModel)
        }
        
        composable("screen_b") { backStackEntry ->
            val parentEntry = remember(backStackEntry) {
                navController.getBackStackEntry("my_graph")
            }
            val sharedViewModel: SharedViewModel = hiltViewModel(parentEntry)
            
            ScreenB(navController, sharedViewModel)
        }
    }
}
```

## Animaciones de Transición

### 1. Transiciones Personalizadas

```kotlin
// Con accompanist-navigation-animation (deprecado en favor de Compose 1.7+)
composable(
    route = "details",
    enterTransition = {
        slideIntoContainer(
            towards = AnimatedContentTransitionScope.SlideDirection.Left,
            animationSpec = tween(300)
        )
    },
    exitTransition = {
        slideOutOfContainer(
            towards = AnimatedContentTransitionScope.SlideDirection.Left,
            animationSpec = tween(300)
        )
    },
    popEnterTransition = {
        slideIntoContainer(
            towards = AnimatedContentTransitionScope.SlideDirection.Right,
            animationSpec = tween(300)
        )
    },
    popExitTransition = {
        slideOutOfContainer(
            towards = AnimatedContentTransitionScope.SlideDirection.Right,
            animationSpec = tween(300)
        )
    }
) {
    DetailsScreen()
}
```

### 2. Animaciones Condicionales

```kotlin
composable(
    route = "details/{id}",
    enterTransition = {
        when (initialState.destination.route) {
            "home" -> slideInHorizontally { it }
            "list" -> slideInVertically { it }
            else -> fadeIn()
        }
    }
) { backStackEntry ->
    val id = backStackEntry.arguments?.getString("id")
    DetailsScreen(id = id ?: "")
}
```

## Testing de Navigation

### 1. Test de Navegación Básica

```kotlin
@Test
fun testNavigation() {
    val navController = TestNavHostController(ApplicationProvider.getApplicationContext())
    
    composeTestRule.setContent {
        navController.navigatorProvider.addNavigator(ComposeNavigator())
        NavHost(navController = navController, startDestination = "home") {
            composable("home") { HomeScreen(navController) }
            composable("details") { DetailsScreen() }
        }
    }
    
    // Verificar ruta inicial
    assertEquals("home", navController.currentBackStackEntry?.destination?.route)
    
    // Navegar
    composeTestRule.onNodeWithText("Go to Details").performClick()
    
    // Verificar nueva ruta
    assertEquals("details", navController.currentBackStackEntry?.destination?.route)
}
```

### 2. Test con Argumentos

```kotlin
@Test
fun testNavigationWithArgs() {
    val navController = TestNavHostController(ApplicationProvider.getApplicationContext())
    
    composeTestRule.setContent {
        navController.navigatorProvider.addNavigator(ComposeNavigator())
        NavHost(navController = navController, startDestination = "home") {
            composable("home") {
                HomeScreen(
                    onItemClick = { id -> navController.navigate("details/$id") }
                )
            }
            composable(
                route = "details/{id}",
                arguments = listOf(navArgument("id") { type = NavType.StringType })
            ) { backStackEntry ->
                val id = backStackEntry.arguments?.getString("id")
                DetailsScreen(id = id ?: "")
            }
        }
    }
    
    composeTestRule.onNodeWithText("Item 123").performClick()
    
    assertEquals("details/123", navController.currentBackStackEntry?.destination?.route)
}
```

## Mejores Prácticas

### 1. Separar Lógica de Navegación

```kotlin
// ❌ MALO - NavController en ViewModel
class HomeViewModel(
    private val navController: NavController  // ¡NO!
) : ViewModel() {
    fun onItemClick(id: String) {
        navController.navigate("details/$id")
    }
}

// ✅ BUENO - Events
class HomeViewModel : ViewModel() {
    private val _navigationEvent = Channel<NavigationEvent>()
    val navigationEvent = _navigationEvent.receiveAsFlow()
    
    fun onItemClick(id: String) {
        viewModelScope.launch {
            _navigationEvent.send(NavigationEvent.NavigateToDetails(id))
        }
    }
}

sealed class NavigationEvent {
    data class NavigateToDetails(val id: String) : NavigationEvent()
}

@Composable
fun HomeScreen(
    viewModel: HomeViewModel = hiltViewModel(),
    navController: NavController
) {
    LaunchedEffect(Unit) {
        viewModel.navigationEvent.collect { event ->
            when (event) {
                is NavigationEvent.NavigateToDetails -> {
                    navController.navigate("details/${event.id}")
                }
            }
        }
    }
    
    HomeContent(onItemClick = viewModel::onItemClick)
}
```

### 2. Type-Safe Navigation

```kotlin
// Usar sealed class para rutas
sealed class Route(val route: String) {
    object Home : Route("home")
    object Profile : Route("profile")
    data class Details(val id: String = "{id}") : Route("details/{id}") {
        fun createRoute(id: String) = "details/$id"
    }
}

// Extensión para navegar de forma segura
fun NavController.navigateSafe(route: Route) {
    navigate(route.route)
}
```

### 3. Manejar System Back

```kotlin
@Composable
fun MyScreen(navController: NavController) {
    BackHandler(enabled = shouldInterceptBack) {
        // Manejar back personalizado
        showExitDialog()
    }
}
```

## Recursos

- [Navigation Compose](https://developer.android.com/jetpack/compose/navigation)
- [Navigation Testing](https://developer.android.com/jetpack/compose/navigation-testing)
- [Deep Links](https://developer.android.com/guide/navigation/navigation-deep-link)
- [Type Safety](https://developer.android.com/guide/navigation/navigation-type-safety)

---
*La navegación en Compose debe ser declarativa y desacoplada de la lógica de negocio*
