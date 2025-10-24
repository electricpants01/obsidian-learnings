
Primero en el libs.versions.toml, tiene muchas dependencias, pero no las usa todas, 
### libraries

- Usan Material 3
- compose ui test
- androidx-core-splashscreen
- androidx-glance-appwidget
- androidx-compose-bom // dependencias de compose
- room // base de datos 
- espreso // ui test
- Coil // para mostrar imagenes
- robolectric // para ui test usando la JVM y no un dispositivo 
- roborazzi, roborazzi-compose
- rometools-modules 
- horologist-audio-ui
- hilt // com.google.dagger:hilt-android, no veo dependencias de dagger, verificar si solo se necesita traer hilt y ya no dagger, o sea que hilt ya viene por defecto (dagger-hilt)

### plugins 

- parcelize
- ksp
- spotless
- version-catalog-update
- secrets // com.google.android.libraries.mapsplatform.secrets-gradle-plugin, me imagino que es la dependencias para guardar los APIS o secrets