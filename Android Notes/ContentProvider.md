# ContentProvider

## Implementación

```kotlin
class MyContentProvider : ContentProvider() {
    override fun onCreate(): Boolean = true
    
    override fun query(uri: Uri, projection: Array<String>?, selection: String?,
                      selectionArgs: Array<String>?, sortOrder: String?): Cursor? {
        // Query data
        return null
    }
    
    override fun insert(uri: Uri, values: ContentValues?): Uri? = null
    override fun update(uri: Uri, values: ContentValues?, selection: String?,
                       selectionArgs: Array<String>?): Int = 0
    override fun delete(uri: Uri, selection: String?, selectionArgs: Array<String>?): Int = 0
    override fun getType(uri: Uri): String? = null
}
```

## Recursos
- [ContentProvider](https://developer.android.com/guide/topics/providers/content-providers)
