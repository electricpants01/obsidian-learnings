sacado de https://github.com/android/compose-samples/tree/main/Jetcaster

Mi idea principal es identificar los patrones que se ocupan en la app, estrategias para identificar:
- observar el build.gradle // ver que dependencias guarda
- observar el MainActivity // activity o fragment de la pantalla
- observer el ViewModel // ver como usar Flow, StateFlow, SharedFlow
- observar los unit tests // usan Mockito o MockK
- observaciones generales // que componentes nuevos usan o yo no se que existen, etc

## Generales 

Usan `SharedTransitionLayout`, me imagino que se usan para tener una mejor transicion al navegar de una vista a otra

`CompositionLocalProvider`, puedes crear tus propios scopes, pero no se crear mis propios scopes. Lo he visto en ebay, lo ocupan para renderear imagenes en @Preview, y en los unit tests bastantes, pero no he indagado aparte que beneficios tiene de usarlos 

`AlertDialog`, viene de Material 3, pero no se que hace, me imagino que es un popup en donde sale botones como `cancel` y `accept`.

He visto que este proyecto, tambien tiene codigo para wear (relojes inteligentes) y tv (una app para una smart tv) son modulos, pero no para una app mobile

## Dependencies
[[Dependencies]]

## Modules
[[Modules]]

