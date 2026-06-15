# Digital Collections - Deeplink System Documentation  
  
> 📚 Comprehensive guide to understanding deeplink flows and architecture in the Digital Collections module  
  
## Table of Contents  
- [Overview](#overview)  
- [Architecture](#architecture)  
- [File Structure](#file-structure)  
- [Deeplink Types](#deeplink-types)  
- [Processing Flows](#processing-flows)  
- [Testing](#testing)  
- [Troubleshooting](#troubleshooting)  
  
---  
  
## Overview  
  
The Digital Collections module supports two types of deeplinks:  
1. **HTTP/HTTPS deeplinks** - Standard web URLs (e.g., `https://www.ebay.com/collection/hub`)  
2. **Custom scheme deeplinks** - App-specific URLs (e.g., `ebay://link?nav=user.priceCards`)  
  
### Key Features  
✅ Automatic authentication handling    
✅ Fallback to Hub page on invalid links    
✅ Source tracking for analytics    
✅ Feature toggle support    
✅ Multiple entry points (Hub, Categories, Card Scan, Repacks)  
  
---  
  
## Architecture  
  
```  
┌─────────────────────────────────────────────────────────────┐  
│                    Deeplink Entry Points                     │  
├─────────────────────────────────────────────────────────────┤  
│                                                               │  
│  HTTP/HTTPS Deeplinks              Custom Scheme (ebay://)  │  
│  └─> DigitalCollectionDeepLinkActivity   └─> LinkProcessors │  
│                                                               │  
└─────────────────────────────────────────────────────────────┘  
                            │                            ▼┌─────────────────────────────────────────────────────────────┐  
│                   Processing & Validation                    │  
├─────────────────────────────────────────────────────────────┤  
│                                                               │  
│  • DeepLinkChecker - Validates deeplink format              │  
│  • DigitalCollectionDeepLinkIntentHelper - Parses & routes  │  
│  • LinkProcessors - Handle custom scheme links              │  
│                                                               │  
└─────────────────────────────────────────────────────────────┘  
                            │                            ▼┌─────────────────────────────────────────────────────────────┐  
│                   Authentication Check                       │  
├─────────────────────────────────────────────────────────────┤  
│                                                               │  
│  CollectibleDeepLink {                                       │  
│    intent: Intent                                            │  
│    isSignInRequired: Boolean                                 │  
│  }                                                            │  
│                                                               │  
│  If sign-in required → SignInFactory                         │  
│  Else → Launch intent directly                               │  
│                                                               │  
└─────────────────────────────────────────────────────────────┘  
                            │                            ▼┌─────────────────────────────────────────────────────────────┐  
│                  Launch Target Activity                      │  
├─────────────────────────────────────────────────────────────┤  
│                                                               │  
│  • DigitalCollectionsActivity (Hub, Categories)             │  
│  • Card Scan Activity (CITA)                                │  
│  • Repacks Screen                                            │  
│                                                               │  
└─────────────────────────────────────────────────────────────┘  
```  
  
---  
  
## File Structure  
  
### 📁 Core Files  
  
#### 1. Entry Point  
```kotlin  
// Path: digitalCollections/digitalCollectionsImpl/src/main/java/com/ebay/mobile/digitalcollections/impl/view/  
DigitalCollectionDeepLinkActivity.kt  
```  
**Purpose**: Landing activity for all `/collection/` HTTP deeplinks  
- Receives deeplink intent  
- Delegates to `DigitalCollectionDeepLinkIntentHelper`  
- Handles authentication flow  
- Launches target activity  
  
**Key Methods**:  
- `onCreate()` - Entry point, processes deeplink  
  
---  
  
#### 2. Helper & Router  
```kotlin  
// Path: digitalCollections/digitalCollectionsImpl/src/main/java/com/ebay/mobile/digitalcollections/impl/helper/  
DigitalCollectionDeepLinkIntentHelper.kt  
```  
**Purpose**: Validates and routes HTTP/HTTPS deeplinks  
- Parses URL path segments  
- Validates deeplink format  
- Creates appropriate Intent  
- Logs errors to analytics  
  
**Key Methods**:  
- `checkAndGetCollectionDeepLinkIntent(context, intent)` - Main routing logic  
- `buildFallbackCollectibleActivityIntent()` - Error fallback  
  
**Supported Paths**:  
- `/collection/hub` → Hub page  
- `/collection/category?notionalTypeId=X` → Category page  
- `/collection/image-scan` → Card scanner  
- `/repacks` → Repacks page  
  
---  
  
#### 3. Link Processors (Custom Scheme)  
  
##### CitaCardScanDeepLinkProcessor.kt  
```kotlin  
// Path: digitalCollections/digitalCollectionsImpl/src/main/java/com/ebay/mobile/digitalcollections/impl/deeplink/  
CitaCardScanDeepLinkProcessor.kt  
```  
**Purpose**: Handles card scanning deeplinks  
- Format: `ebay://link?nav=user.priceCards&src=<source>`  
- Routes to Card Scan activity  
- Tracks source for analytics  
  
**Sources**:  
- `src=home` → HOME launch source  
- `src=myebay` → MY_EBAY launch source    
- Other/empty → MY_COLLECTION launch source  
  
---  
  
##### RepacksLinkProcessor.kt  
```kotlin  
// Path: digitalCollections/digitalCollectionsImpl/src/main/java/com/ebay/mobile/digitalcollections/impl/deeplink/  
RepacksLinkProcessor.kt  
```  
**Purpose**: Handles repacks deeplinks  
- Format: `ebay://link/?nav=repacks`  
- No authentication required  
- Direct launch to Repacks screen  
  
---  
  
##### CollectiblesVaultLinkProcessor.kt  
```kotlin  
// Path: digitalCollections/digitalCollectionsImpl/src/main/java/com/ebay/mobile/digitalcollections/impl/helper/  
CollectiblesVaultLinkProcessor.kt  
```  
**Purpose**: Handles vault/category deeplinks  
- Format: `ebay://link?nav=user.collectibles.category&notionalTypeId=X`  
- Requires `notionalTypeId` parameter  
- Launches category view with specified type  
  
---  
  
#### 4. Configuration  
  
##### Dagger Module  
```kotlin  
// Path: digitalCollections/digitalCollectionsImpl/src/main/java/com/ebay/mobile/digitalcollections/impl/dagger/  
DigitalCollectionsApplicationModule.kt  
```  
**Purpose**: Dependency injection setup  
- Registers `DigitalCollectionDeepLinkActivity`  
- Binds LinkProcessors to navigation targets  
- Provides feature toggle configurations  
  
---  
  
##### Feature Toggles  
```kotlin  
// Path: digitalCollections/digitalCollectionsImpl/src/main/java/com/ebay/mobile/digitalcollections/impl/  
DigitalCollectionsFeatureToggles.kt  
```  
**Toggles**:  
- `CITA_DEEP_LINK` - Enable/disable CITA card scan deeplinks  
  
---  
  
## Deeplink Types  
  
### 🌐 HTTP/HTTPS Deeplinks  
  
All handled by `DigitalCollectionDeepLinkActivity` → `DigitalCollectionDeepLinkIntentHelper`  
  
#### 1. Hub/Home  
```  
URL: https://www.ebay.com/collection/hub  
  
Authentication: ✅ RequiredTarget: Digital Collections Hub page  
```  
  
#### 2. Category  
```  
URL: https://www.ebay.com/collection/category?notionalTypeId={ID}  
  
Parameters:  
  - notionalTypeId: String (required) - Category identifier  
Authentication: ✅ RequiredTarget: Specific category view  
```  
  
#### 3. Image/Card Scan  
```  
URL: https://www.ebay.com/collection/image-scan  
  
Authentication: ✅ RequiredTarget: Card scanning activity  
Source: HOME  
```  
  
#### 4. Repacks  
```  
URL: https://www.ebay.com/repacks  
  
Authentication: ❌ Not RequiredTarget: Repacks screen  
```  
  
---  
  
### 📱 Custom Scheme Deeplinks (ebay://)  
  
Each handled by a specific `LinkProcessor`  
  
#### 1. Card Scan/Pricing  
```  
Format: ebay://link?nav=user.priceCards&src={source}  
  
Processor: CitaCardScanDeepLinkProcessor  
NAV_TARGET: "user.priceCards"  
  
Parameters:  
  - nav: "user.priceCards" (case-insensitive, required)  - src: String (optional)    • "home" → CardScanLaunchSource.HOME    • "myebay" → CardScanLaunchSource.MY_EBAY    • other/empty → CardScanLaunchSource.MY_COLLECTION  
  
Examples:  
  ebay://link?nav=user.priceCards&src=home  ebay://link?nav=user.priceCards&src=myEbay  ebay://link?nav=user.pricecards```  
  
#### 2. Repacks  
```  
Format: ebay://link/?nav=repacks  
  
Processor: RepacksLinkProcessor  
NAV_TARGET: "repacks"  
  
Parameters:  
  - nav: "repacks" (required)  
Example:  
  ebay://link/?nav=repacks```  
  
#### 3. Vault/Category  
```  
Format: ebay://link?nav=user.collectibles.category&notionalTypeId={ID}  
  
Processor: CollectiblesVaultLinkProcessor  
NAV_TARGET: "user.collectibles.category"  
  
Parameters:  
  - nav: "user.collectibles.category" (required)  - notionalTypeId: String (required)  
Example:  
  ebay://link?nav=user.collectibles.category&notionalTypeId=Vault```  
  
---  
  
## Processing Flows  
  
### Flow 1: HTTP/HTTPS Deeplink Processing  
  
```  
┌─────────────────────────────────────────────────────────┐  
│ 1. User clicks deeplink                                 │  
│    https://www.ebay.com/collection/hub                  │  
└────────────────────┬────────────────────────────────────┘  
                     │                     ▼┌─────────────────────────────────────────────────────────┐  
│ 2. Android System                                       │  
│    • Resolves intent filter                             │  
│    • Launches DigitalCollectionDeepLinkActivity         │  
└────────────────────┬────────────────────────────────────┘  
                     │                     ▼┌─────────────────────────────────────────────────────────┐  
│ 3. DigitalCollectionDeepLinkActivity.onCreate()         │  
│    • Inject dependencies                                │  
│    • Call helper.checkAndGetCollectionDeepLinkIntent()  │  
└────────────────────┬────────────────────────────────────┘  
                     │                     ▼┌─────────────────────────────────────────────────────────┐  
│ 4. DigitalCollectionDeepLinkIntentHelper                │  
│    • Validate with DeepLinkChecker.isDeepLinkIntent()   │  
│    • Parse intent.data.pathSegments                     │  
│    • Match path: "collection" → segment[1]              │  
└────────────────────┬────────────────────────────────────┘  
                     │        ┌────────────┴──────────────┐        │                           │        ▼                           ▼┌──────────────────┐    ┌────────────────────────┐  
│ Valid Deeplink   │    │ Invalid/Malformed      │  
│                  │    │                        │  
│ • Parse params   │    │ • Log error to APLS    │  
│ • Build intent   │    │ • Return fallback      │  
│ • Set auth flag  │    │   (Hub page)           │  
└────────┬─────────┘    └────────────────────────┘  
         │         ▼┌─────────────────────────────────────────────────────────┐  
│ 5. Return CollectibleDeepLink                           │  
│    data class CollectibleDeepLink(                      │  
│      intent: Intent,                                    │  
│      isSignInRequired: Boolean                          │  
│    )                                                    │  
└────────────────────┬────────────────────────────────────┘  
                     │        ┌────────────┴──────────────┐        │                           │        ▼                           ▼┌──────────────────┐    ┌────────────────────────┐  
│ Sign-In NOT      │    │ Sign-In REQUIRED       │  
│ Required         │    │                        │  
│                  │    │ • Check user state     │  
│ • Launch intent  │    │ • If signed in →       │  
│ • finish()       │    │   launch intent        │  
└──────────────────┘    │ • If not signed in →   │  
                        │   launch SignInFactory │                        │ • After sign-in →      │                        │   launch intent        │                        └────────────────────────┘```  
  
---  
  
### Flow 2: Custom Scheme (ebay://) Processing  
  
```  
┌─────────────────────────────────────────────────────────┐  
│ 1. App receives custom scheme deeplink                  │  
│    ebay://link?nav=user.priceCards&src=home             │  
└────────────────────┬────────────────────────────────────┘  
                     │                     ▼┌─────────────────────────────────────────────────────────┐  
│ 2. Android System                                       │  
│    • Matches intent filter for "ebay" scheme            │  
│    • Routes to LinkProcessor system                     │  
└────────────────────┬────────────────────────────────────┘  
                     │                     ▼┌─────────────────────────────────────────────────────────┐  
│ 3. LinkProcessor Resolution                             │  
│    • Extract "nav" query parameter                      │  
│    • Map to registered LinkProcessor via Dagger         │  
│      @StringKey annotation                              │  
└────────────────────┬────────────────────────────────────┘  
                     │        ┌────────────┴──────────────┬──────────────────┐        │                           │                  │        ▼                           ▼                  ▼┌──────────────────┐    ┌────────────────┐  ┌──────────────────┐  
│ nav=             │    │ nav=repacks    │  │ nav=user.        │  
│ user.priceCards  │    │                │  │ collectibles.    │  
│                  │    │ RepacksLink    │  │ category         │  
│ CitaCardScan     │    │ Processor      │  │                  │  
│ DeepLinkProc.    │    │                │  │ Collectibles     │  
│                  │    │ • Build intent │  │ VaultLink        │  
│ • Parse src      │    │   for Repacks  │  │ Processor        │  
│ • Map to launch  │    │ • Return       │  │                  │  
│   source         │    │                │  │ • Require        │  
│ • Build card     │    │                │  │   notionalTypeId │  
│   scan intent    │    │                │  │ • Build category │  
│ • Return         │    │                │  │   intent         │  
│                  │    │                │  │ • Return         │  
└────────┬─────────┘    └────────┬───────┘  └────────┬─────────┘  
         │                       │                   │         └───────────────────────┴───────────────────┘                                 │                                 ▼┌─────────────────────────────────────────────────────────┐  
│ 4. System launches returned Intent                      │  
│    • Start target activity                              │  
│    • Pass parameters via intent extras                  │  
└─────────────────────────────────────────────────────────┘  
```  
  
---  
  
### Flow 3: Authentication Handling  
  
```  
┌─────────────────────────────────────────────────────────┐  
│ CollectibleDeepLink received                            │  
│ isSignInRequired = true/false                           │  
└────────────────────┬────────────────────────────────────┘  
                     │        ┌────────────┴──────────────┐        │                           │        ▼                           ▼┌──────────────────┐    ┌────────────────────────────────┐  
│ isSignInRequired │    │ isSignInRequired = true        │  
│ = false          │    │                                │  
│                  │    │ 1. Launch coroutine            │  
│ Direct launch    │    │    lifecycleScope.launch {     │  
│ startActivity()  │    │      repeatOnLifecycle(        │  
│ finish()         │    │        Lifecycle.State.CREATED │  
│                  │    │      ) {                       │  
└──────────────────┘    │        // Check auth state     │  
                        │      }                         │                        │    }                           │                        │                                │                        │ 2. Collect user state          │                        │    currentUserState            │                        │      .isSignedInFlow           │                        │      .collect { signedIn ->    │                        └────────────┬───────────────────┘                                     │                        ┌────────────┴──────────────┐                        │                           │                        ▼                           ▼                ┌──────────────┐          ┌──────────────────┐                │ User signed  │          │ User NOT signed  │                │ in           │          │ in               │                │              │          │                  │                │ • Launch     │          │ • Create sign-in │                │   intent     │          │   launcher       │                │ • finish()   │          │ • Launch         │                └──────────────┘          │   SignInFactory  │                                          │ • On result →    │                                          │   launch intent  │                                          │ • finish()       │                                          └──────────────────┘```  
  
---  
  
## Testing  
  
### 🧪 ADB Commands for Testing  
  
#### HTTP/HTTPS Deeplinks  
```bash  
# Hub page  
adb shell am start -W -a android.intent.action.VIEW \  
  -d "https://www.ebay.com/collection/hub"  
# Category with notionalTypeId  
adb shell am start -W -a android.intent.action.VIEW \  
  -d "https://www.ebay.com/collection/category?notionalTypeId=Vault"  
# Image scan  
adb shell am start -W -a android.intent.action.VIEW \  
  -d "https://www.ebay.com/collection/image-scan"  
# Repacks  
adb shell am start -W -a android.intent.action.VIEW \  
  -d "https://www.ebay.com/repacks"  
```  
  
#### Custom Scheme Deeplinks  
```bash  
# Card scan from home  
adb shell am start -W -a android.intent.action.VIEW \  
  -d "ebay://link?nav=user.priceCards&src=home"  
# Card scan from MyEbay  
adb shell am start -W -a android.intent.action.VIEW \  
  -d "ebay://link?nav=user.priceCards&src=myEbay"  
# Card scan (no source)  
adb shell am start -W -a android.intent.action.VIEW \  
  -d "ebay://link?nav=user.pricecards"  
# Repacks  
adb shell am start -W -a android.intent.action.VIEW \  
  -d "ebay://link/?nav=repacks"  
# Vault category  
adb shell am start -W -a android.intent.action.VIEW \  
  -d "ebay://link?nav=user.collectibles.category&notionalTypeId=Vault"  
```  
  
### 📝 Unit Tests Location  
  
```  
digitalCollections/digitalCollectionsImpl/src/test/java/com/ebay/mobile/digitalcollections/impl/  
  
Tests:  
├── deeplink/  
│   ├── CitaCardScanDeepLinkProcessorTest.kt  
│   └── RepacksLinkProcessorTest.kt  
└── helper/  
    └── DigitalCollectionDeeplinkIntentHelperTest.kt```  
  
---  
  
## Troubleshooting  
  
### Common Issues & Solutions  
  
#### ❌ Deeplink not working  
**Check**:  
1. Verify intent filter in AndroidManifest  
2. Check if `DeepLinkChecker.isDeepLinkIntent()` returns true  
3. Verify path segments match expected format  
4. Check logs for `DeepLinkTracker` error messages  
  
#### ❌ Authentication loop  
**Check**:  
1. Verify `isSignInRequired` flag is set correctly  
2. Check `currentUserState.isSignedInFlow` is emitting correctly  
3. Ensure `finish()` is called after launching intent  
  
#### ❌ Wrong screen launched  
**Check**:  
1. Path segment parsing in `DigitalCollectionDeepLinkIntentHelper`  
2. LinkProcessor registration in Dagger module  
3. NAV_TARGET constant matches query parameter  
  
#### ❌ Feature toggle issues  
**Check**:  
1. `CITA_DEEP_LINK` toggle value  
2. Toggle check in `DigitalCollectionsFactoryImpl`  
  
---  
  
## Key Takeaways  
  
### 💡 Best Practices  
  
1. **Always validate deeplinks**  
   - Use `DeepLinkChecker` before processing  
   - Provide fallback for invalid links  
  
2. **Handle authentication gracefully**  
   - Check sign-in requirements  
   - Use coroutines for async auth checks  
   - Always finish() activity after launching  
  
3. **Track analytics**  
   - Log successful deeplink launches  
   - Log errors with `DeepLinkTracker`  
   - Include source tracking for attribution  
  
4. **Test thoroughly**  
   - Test both signed-in and signed-out states  
   - Test with and without required parameters  
   - Test malformed URLs  
  
### 🔐 Security Considerations  
  
- Validate all query parameters  
- Sanitize user input before using in intents  
- Require authentication for sensitive features  
- Log suspicious deeplink patterns  
  
---  
  
## Related Documentation  
  
- [Universal Link Processing](../universallink/)  
- [Deep Linking Framework](../deeplinking/)  
- [Digital Collections Factory](./digitalCollectionsImpl/src/main/java/com/ebay/mobile/digitalcollections/impl/DigitalCollectionsFactoryImpl.kt)  
  
---  
  
**Last Updated**: December 2025    
**Maintained by**: Digital Collections Team