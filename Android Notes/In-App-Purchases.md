# In-App Purchases

## Google Play Billing

```kotlin
dependencies {
    implementation("com.android.billingclient:billing-ktx:6.1.0")
}
```

## Setup

```kotlin
val billingClient = BillingClient.newBuilder(context)
    .setListener { billingResult, purchases ->
        if (billingResult.responseCode == BillingClient.BillingResponseCode.OK && purchases != null) {
            purchases.forEach { handlePurchase(it) }
        }
    }
    .enablePendingPurchases()
    .build()

billingClient.startConnection(object : BillingClientStateListener {
    override fun onBillingSetupFinished(billingResult: BillingResult) {
        if (billingResult.responseCode == BillingClient.BillingResponseCode.OK) {
            // Ready
        }
    }
    override fun onBillingServiceDisconnected() {}
})
```

## Purchase

```kotlin
val productDetailsParams = QueryProductDetailsParams.newBuilder()
    .setProductList(listOf(
        QueryProductDetailsParams.Product.newBuilder()
            .setProductId("premium")
            .setProductType(BillingClient.ProductType.SUBS)
            .build()
    ))
    .build()

billingClient.queryProductDetailsAsync(productDetailsParams) { billingResult, productDetailsList ->
    // Show products
}
```

## Recursos
- [Billing](https://developer.android.com/google/play/billing)
