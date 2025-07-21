## Payment SDK (Android, Kotlin)
[![JitPack](https://jitpack.io/v/freedompay-global/merchant-sdk-android.svg)](https://jitpack.io/#freedompay-global/merchant-sdk-android)

The Payment SDK is a library that simplifies interaction with the Freedom Pay API. Supports Android 9 and above.

---
### Table of Contents
- [Getting Started](#getting-started)
- [Installation Instructions](#installation-instructions)
- [SDK integration](#sdk-integration)
- [Integrating the PaymentView (Optional)](#integrating-the-paymentview-optional)
  - [Adding `PaymentView`](#adding-paymentview)
  - [Connecting `PaymentView`](#connecting-paymentview)
  - [Tracking Loading Progress](#tracking-loading-progress-optional)
- [SDK Configuration](#sdk-configuration)
  - [`SdkConfiguration` Overview](#sdkconfiguration-overview)
  - [Applying the Configuration](#applying-the-configuration)
- [Working with the SDK](#working-with-the-sdk)
  - [Create Payment page/frame](#create-payment-pageframe)
  - [Get Payment status](#get-payment-status)
  - [Make Clearing Payment](#make-clearing-payment)
  - [Make Cancel Payment](#make-cancel-payment)
  - [Make Revoke Payment](#make-revoke-payment)
  - [Add New Card](#add-new-card)
  - [Get Added Cards](#get-added-cards)
  - [Remove Added Card](#remove-added-card)
  - [Create Card Payment](#create-card-payment)
  - [Confirm Card Payment](#confirm-card-payment)
  - [Confirm Direct Payment](#confirm-direct-payment)
- [Google Pay Integration](#google-pay-integration)
  - [Create Google Payment](#create-google-payment)
  - [Confirm Google Payment](#confirm-google-payment)
- [Error Handling and Results](#error-handling-and-results)
    - [`FreedomResult.Success<T>`](#freedomresultsuccesst)
    - [`FreedomResult.Error`](#freedomresulterror)
- [Data Structures](#data-structures)
- [Support](#support)
---

### Getting Started

&emsp;Before you begin integrating the **Freedom Payment SDK** into your Android app, ensure you have the following:
- An Android app project with a minimum API level of 28.
- The **Freedom Payment SDK** ZIP file

---
### Installation Instructions

#### 1. Add JitPack repository

Add JitPack to your project's `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
  repositories {
    google()
    mavenCentral()
    maven { url = uri("https://jitpack.io") } // Add this line
  }
}
```

#### 2. Add Dependency
```kotlin
dependencies {
    implementation("com.github.freedompay-global:merchant-sdk-android:${specifyTheLatestVersion}")
}
```

#### 3. Sync the project

---

### SDK integration

#### Initialize

&emsp;To initialize the **Freedom Payment SDK**, call the `create` method of the `FreedomAPI` class. This method requires *three parameters*:
> **NOTE**
> - Your merchant ID
> - Your merchant secret key
> - The payment region

```kotlin
val merchantId = "123456"
val secretKey = "123456789ABCDEF"
val region = Region.KZ
val freedomApi = FreedomAPI.create(merchantId, secretKey, region)
```
##### Table: `Region`
| Enum Constant | Description        |
|---------------|--------------------|
| `KZ`          | Kazakhstan region. |
| `RU`          | Russia region.     |
| `UZ`          | Uzbekistan region. |
| `KG`          | Kyrgyzstan region. |

---

### Integrating the PaymentView (Optional)

#### Adding `PaymentView`

&emsp;Add the PaymentView to your activity's layout file:
 ```xml
<kz.freedompay.paymentsdk.api.view.PaymentView
    android:id="@+id/paymentView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
 ```

#### Connecting `PaymentView`

&emsp;Pass the instance of your PaymentView to the SDK:
```kotlin
freedomApi.setPaymentView(paymentView)
```

#### Tracking Loading Progress (Optional)

&emsp;To track the loading progress of the payment page, use the `onLoadingStateChanged` listener:
```kotlin
paymentView.onLoadingStateChanged { isLoading: Boolean -> 
    // Handle loading state changes (e.g., show/hide a progress indicator)
}
```

---
### SDK Configuration

&emsp;The SDK's behavior is controlled through its configuration, which you manage using the `SdkConfiguration` data class. This class acts as a central container, encapsulating two key components: `UserConfiguration` for customer-specific settings and `OperationalConfiguration` for general operational parameters.

#### `SdkConfiguration` Overview

The `SdkConfiguration` object is passed to the SDK via `freedomApi.setConfiguration()`.

```kotlin
data class SdkConfiguration(
    val userConfiguration: UserConfiguration = UserConfiguration(),
    val operationalConfiguration: OperationalConfiguration = OperationalConfiguration(),
)
```

##### Table: `UserConfiguration`
&emsp;This data class holds customer-specific details.

| Property           | Type      | Description                                                                                                                        | Default Value |
|--------------------|-----------|------------------------------------------------------------------------------------------------------------------------------------|---------------|
| `userPhone`        | `String?` | Customer's phone number. If provided, it will be displayed on the payment page. If `null`, the user will be prompted to enter it.  | `null`        |
| `userContactEmail` | `String?` | Customer's contact email.                                                                                                          | `null`        |
| `userEmail`        | `String?` | Customer's email address. If provided, it will be displayed on the payment page. If `null`, the user will be prompted to enter it. | `null`        |

##### Table: `OperationalConfiguration`
&emsp;This data class contains general operational settings for the SDK.

| Property        | Type          | Description                                                                                                                                                    | Default Value                                                                   |
|-----------------|---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------|
| `testingMode`   | `Boolean?`    | Enables or disables test mode.                                                                                                                                 | `null`                                                                          |
| `language`      | `Language?`   | Sets the language of the payment page. See [`Language`](#table-language) enum for available options.                                                           | `null`                                                                          |
| `lifetime`      | `Int`         | Duration, in seconds, for which the payment page remains valid for completing a payment.                                                                       | `1800`                                                                          |
| `autoClearing`  | `Boolean?`    | Activates automatic clearing of payments.                                                                                                                      | `null`                                                                          |
| `checkUrl`      | `String?`     | URL to which the payment check will be sent.                                                                                                                   | `null`                                                                          |
| `resultUrl`     | `String?`     | URL to which the payment result will be sent.                                                                                                                  | `null`                                                                          |
| `requestMethod` | `HttpMethod?` | HTTP method used for requests to `checkUrl` or `resultUrl`. See [`HttpMethod`](#table-httpmethod) enum. Defaults to `GET` if `checkUrl` or `resultUrl` is set. | `HttpMethod.GET` (if `checkUrl` or `resultUrl` is not `null`, otherwise `null`) |

##### Table: `Language`
| Enum Constant | Description                      |
|---------------|----------------------------------|
| `KZ`          | SDK uses the `Kazakh` language.  |
| `RU`          | SDK uses the `Russian` language. |
| `EN`          | SDK uses the `English` language. |

##### Table: `HttpMethod`
| Enum Constant | Description                      |
|---------------|----------------------------------|
| `GET`         | SDK uses the HTTP `GET` method.  |
| `POST`        | SDK uses the HTTP `POST` method. |

#### Applying the Configuration
&emsp;To apply your desired SDK configuration, create an instance of `SdkConfiguration` and pass it to the `freedomApi.setConfiguration()` method:

```kotlin
val myUserConfig = UserConfiguration(
    userPhone = "1234567890",
    userEmail = "user@example.com"
)

val myOperationalConfig = OperationalConfiguration(
    testingMode = false,
    language = Language.EN, // Using the provided Language enum
    lifetime = 600, // 10 minutes
    autoClearing = true,
    resultUrl = "https://example.com/result",
    requestMethod = HttpMethod.POST // Using the provided HttpMethod enum
)

val sdkConfiguration = SdkConfiguration(
    userConfiguration = myUserConfig,
    operationalConfiguration = myOperationalConfig
)

freedomApi.setConfiguration(sdkConfiguration)
```

---
### Working with the SDK

&emsp;This section details the primary methods available in the SDK for managing payments within your application.

#### Create Payment page/frame
&emsp;To initiate a new payment transaction and display the payment page, call the `createPaymentPage` or `createPaymentFrame` method on your `FreedomAPI` instance. This method handles the presentation of the payment page/frame within the previously configured `PaymentView`.

> **WARNING**
> These methods require that a `PaymentView` has been set using [`freedomApi.setPaymentView()`](#integrating-the-paymentview-optional) prior to calling them.

&emsp;These methods accept several parameters to define the payment details:

| Parameter        | Type                                       | Description                                                                                                                                   | Constraints/Notes |
|------------------|--------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|-------------------|
| `paymentRequest` | `StandardPaymentRequest`                   | Essential details required to initiate a new payment. See [`StandardPaymentRequest`](#standardpaymentrequest-structure) model.                |                   |
| `onResult`       | `(FreedomResult<PaymentResponse>) -> Unit` | Callback function that will be invoked upon the completion of the payment process. See [`PaymentResponse`](#paymentresponse-structure) model. |                   |

&emsp;The process returns an [`FreedomResult<PaymentResponse>`](#error-handling-and-results) object, which can be either:
- **Success**: Contains a `PaymentResponse` object.
- **Error**: Specifies the type of error that occurred.

```kotlin
val paymentRequest = StandardPaymentRequest(
    amount = 123.45f,
    currency = Currency.KZT, // Using the provided Currency enum
    description = "Monthly Subscription",
    userId = "user12345",
    orderId = "SUB-2025-001",
    extraParams = hashMapOf("plan" to "premium")
)
freedomApi.createPaymentPage(
    paymentRequest = paymentRequest
) { result: FreedomResult<PaymentResponse> ->
    when (result) {
        is FreedomResult.Success -> {
            // Payment page processed successfully.
        }
        is FreedomResult.Error -> {
            // An error occurred during the payment process.
        }
    }
}
```

#### Get Payment status
&emsp;To retrieve the current status of a previously initiated payment, use the `getPaymentStatus` method.

&emsp;This method takes these parameters:

| Parameter   | Type                              | Description                                                                                                            | Constraints/Notes |
|-------------|-----------------------------------|------------------------------------------------------------------------------------------------------------------------|-------------------|
| `paymentId` | `Long`                            | Unique identifier of the payment you want to check.                                                                    |                   |
| `onResult`  | `(FreedomResult<Status>) -> Unit` | Callback function that will be invoked with the result of the payment status. See [`Status`](#status-structure) model. |                   |

&emsp;The process returns an [`FreedomResult<Status>`](#error-handling-and-results) object, which can be either:
- **Success**: Contains a `Status` object.
- **Error**: Specifies the type of error that occurred.

```kotlin
freedomApi.getPaymentStatus(paymentId = 123456L) { result: FreedomResult<Status> ->
    when (result) {
        is FreedomResult.Success -> {
            // Payment status retrieved successfully.
        }
        is FreedomResult.Error -> {
            // Failed to retrieve payment status.
        }
    }
}
```

#### Make Clearing Payment
> **NOTE**
> This method is specifically designed for merchants who have **auto-clearing disabled** in their SDK configuration. Auto-clearing can be managed via the `autoClearing` property within the [`OperationalConfiguration`](#table-operationalconfiguration) of your `SdkConfiguration`.

&emsp;Use the `makeClearingPayment` method to explicitly initiate the clearing (final capture) of funds for a previously authorized payment. This method gives you the flexibility to clear an amount that may be different from the original amount specified when the payment was created (e.g., for partial captures).

&emsp;This method takes these parameters:

| Parameter   | Type                                      | Description                                                                                                                                | Constraints/Notes                                                                                                      |
|-------------|-------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------|
| `paymentId` | `Long`                                    | Unique identifier of the payment you want to clear.                                                                                        |                                                                                                                        |
| `amount`    | `Float?`                                  | Amount to be cleared. If `null`, the full amount of the original authorized payment will be cleared.                                       | Optional. Defaults to null. Must be between `0.01` and `99999999.00` if provided. Cannot exceed the authorized amount. |
| `onResult`  | `(FreedomResult<ClearingStatus>) -> Unit` | Callback function that will be invoked with the result of the clearing operation. See [`ClearingStatus`](#clearingstatus-structure) model. |                                                                                                                        |

&emsp;The process returns an [`FreedomResult<ClearingStatus>`](#error-handling-and-results) object, which can be either:
- **Success**: Contains a `ClearingStatus` object.
- **Error**: Specifies the type of error that occurred.

```kotlin
freedomApi.makeClearingPayment(paymentId = 123456L) { result: FreedomResult<ClearingStatus> ->
    when (result) {
        is FreedomResult.Success -> {
            // Handle the clearing status.
        }
        is FreedomResult.Error -> {
            // Failed to clear the payment.
        }
    }
}
```

#### Make Cancel Payment
> **NOTE**
> This method is specifically designed for merchants who have **auto-clearing disabled** in their SDK configuration. Auto-clearing can be managed via the `autoClearing` property within the [`OperationalConfiguration`](#table-operationalconfiguration) of your `SdkConfiguration`.

&emsp;Use the `makeCancelPayment` method to reverse an authorized payment, effectively unblocking the amount on the customer's card. This ensures that the funds will not be charged.

&emsp;This method takes these parameters:

| Parameter   | Type                                       | Description                                                                                                                                    | Constraints/Notes |
|-------------|--------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|-------------------|
| `paymentId` | `Long`                                     | Unique identifier of the payment you want to cancel.                                                                                           |                   |
| `onResult`  | `(FreedomResult<PaymentResponse>) -> Unit` | Callback function that will be invoked with the result of the cancellation attempt. See [`PaymentResponse`](#paymentresponse-structure) model. |                   |

&emsp;The process returns an [`FreedomResult<PaymentResponse>`](#error-handling-and-results) object, which can be either:
- **Success**: Contains a `PaymentResponse` object.
- **Error**: Specifies the type of error that occurred.

```kotlin
freedomApi.makeCancelPayment(paymentId = 123456L) { result: FreedomResult<PaymentResponse> ->
    when (result) {
        is FreedomResult.Success -> {
            // Cancellation attempt completed successfully.
        }
        is FreedomResult.Error -> {
            // Failed to cancel the payment.
        }
    }
}
```

#### Make Revoke Payment
&emsp;The `makeRevokePayment` method allows you to process a full or partial refund for a completed payment.

&emsp;This method takes these parameters:

| Parameter   | Type                                       | Description                                                                                                                              | Constraints/Notes                                                                                                      |
|-------------|--------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------|
| `paymentId` | `Long`                                     | Unique identifier of the payment you want to revoke (refund).                                                                            |                                                                                                                        |
| `amount`    | `Float?`                                   | Amount of funds to return. If `null`, the full amount of the original debited payment will be refunded.                                  | Optional. Defaults to null. Must be between `0.01` and `99999999.00` if provided. Cannot exceed the authorized amount. |
| `onResult`  | `(FreedomResult<PaymentResponse>) -> Unit` | Callback function that will be invoked with the result of the refund process. See [`PaymentResponse`](#paymentresponse-structure) model. |                                                                                                                        |

&emsp;The process returns an [`FreedomResult<PaymentResponse>`](#error-handling-and-results) object, which can be either:
- **Success**: Contains a `PaymentResponse` object.
- **Error**: Specifies the type of error that occurred.

```kotlin
freedomApi.makeRevokePayment(paymentId = 123456L) { result: FreedomResult<PaymentResponse> ->
    when (result) {
        is FreedomResult.Success -> {
            // Refund process completed successfully.
        }
        is FreedomResult.Error -> {
            // Failed to process the refund.
        }
    }
}
```

#### Add New Card
&emsp;The `addNewCard` method facilitates the secure tokenization and addition of a new payment card to a customer's profile. This process allows future payments to be made without requiring the customer to re-enter their card details.

> **WARNING**
> This method requires that a `PaymentView` has been set using [`freedomApi.setPaymentView()`](#integrating-the-paymentview-optional) prior to calling it, as it will display a web-based form within the `PaymentView` for the customer to securely enter their card details.

&emsp;This method takes these parameters:

| Parameter  | Type                                       | Description                                                                                                                                         | Constraints/Notes                                                        |
|------------|--------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| `userId`   | `String`                                   | Identifier for a user to associate this new card with.                                                                                              | Must contain 1-50 characters. Also, must match regex `^[a-zA-Z0-9_-]+$`. |
| `orderId`  | `String?`                                  | Unique identifier for this order within your system.                                                                                                | Optional. Defaults to null. Must contain 1-50 characters.                |
| `onResult` | `(FreedomResult<PaymentResponse>) -> Unit` | Callback function that will be invoked upon the completion of the card addition process. See [`PaymentResponse`](#paymentresponse-structure) model. |                                                                          |

&emsp;The process returns an [`FreedomResult<PaymentResponse>`](#error-handling-and-results) object, which can be either:
- **Success**: Contains a `PaymentResponse` object.
- **Error**: Specifies the type of error that occurred.


```kotlin
freedomApi.addNewCard(
    userId = "user12345",
    orderId = "CARDADD-SESSION-XYZ" // Optional tracking ID
) { result: FreedomResult<PaymentResponse> ->
    when (result) {
        is FreedomResult.Success -> {
            // Card addition process completed successfully.
        }
        is FreedomResult.Error -> {
            // Failed to add the card.
        }
    }
}
```
#### Get Added Cards
&emsp;The `getAddedCards` method allows you to retrieve a list of payment cards that have been previously added and tokenized for a specific user. 

&emsp;This method takes these parameters:

| Parameter  | Type                                  | Description                                                                                                                      | Constraints/Notes                                                        |
|------------|---------------------------------------|----------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| `userId`   | `String`                              | Identifier of the user whose added cards you wish to retrieve.                                                                   | Must contain 1-50 characters. Also, must match regex `^[a-zA-Z0-9_-]+$`. |
| `onResult` | `(FreedomResult<List<Card>>) -> Unit` | Callback function that will be invoked upon the completion of the card retrieval operation. See [`Card`](#card-structure) model. |                                                                          |

&emsp;The process returns an [`FreedomResult<List<Card>>`](#error-handling-and-results) object, which can be either:
- **Success**: Contains a list of `Card` objects.
- **Error**: Specifies the type of error that occurred.

```kotlin
freedomApi.getAddedCards(userId = "user12345") { result: FreedomResult<List<Card>> ->
    when (result) {
        is FreedomResult.Success -> {
            // List of added cards retrieved successfully.
        }
        is FreedomResult.Error -> {
            // Failed to retrieve added cards.
        }
    }
}
```

#### Remove Added Card
&emsp;`removeAddedCard` method allows you to securely remove a previously tokenized payment card associated with a specific user.

&emsp;This method takes these parameters:

| Parameter   | Type                                   | Description                                                                                                                                                                      | Constraints/Notes                                                        |
|-------------|----------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| `userId`    | `String`                               | Identifier of the user from whom the card will be removed.                                                                                                                       | Must contain 1-50 characters. Also, must match regex `^[a-zA-Z0-9_-]+$`. |
| `cardToken` | `String`                               | Unique token of the card to be removed. This token is obtained when the card is added (via [`addNewCard`](#add-new-card) or retrieved using [`getAddedCards`](#get-added-cards). | Must contain 1 or more characters.                                       |
| `onResult`  | `(FreedomResult<RemovedCard>) -> Unit` | Callback function that will be invoked upon the completion of the card removal operation. See [`RemovedCard`](#removedcard-structure) model.                                     |                                                                          |

&emsp;The process returns an [`FreedomResult<RemovedCard>`](#error-handling-and-results) object, which can be either:
- **Success**: Contains a `RemovedCard` object.
- **Error**: Specifies the type of error that occurred.

```kotlin
freedomApi.removeAddedCard(
    userId = "user12345",
    cardToken = "card-token-123abc"
) { result: FreedomResult<RemovedCard> ->
    when (result) {
        is FreedomResult.Success -> {
            // Card removal process completed successfully.
        }
        is FreedomResult.Error -> {
            // Failed to remove the card.
        }
    }
}
```
#### Create Card Payment
&emsp;The `createCardPayment` method is used to initiate a payment using a previously tokenized (saved) card.

> **NOTE**
> A payment initiated with `createCardPayment` is a two-stage process. After successfully calling this method, the created payment must be confirmed using either the [`confirmCardPayment`](#confirm-card-payment) or [`confirmDirectPayment`](#confirm-direct-payment) method to finalize the transaction.

&emsp;This method takes these parameters:

| Parameter        | Type                                       | Description                                                                                                                                                     | Constraints/Notes |
|------------------|--------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------|
| `paymentRequest` | `TokenizedPaymentRequest`                  | Essential details required to initiate a new payment with previously tokenized card. See [`TokenizedPaymentRequest`](#tokenizedpaymentrequest-structure) model. |                   |
| `onResult`       | `(FreedomResult<PaymentResponse>) -> Unit` | Callback function that will be invoked with the result of the payment initiation. See [`PaymentResponse`](#paymentresponse-structure) model.                    |                   |

&emsp;The process returns an [`FreedomResult<PaymentResponse>`](#error-handling-and-results) object, which can be either:
- **Success**: Contains a `PaymentResponse` object.
- **Error**: Specifies the type of error that occurred.

```kotlin
val paymentRequest = TokenizedPaymentRequest(
    amount = 123.45f,
    currency = Currency.KZT,
    description = "Monthly Subscription",
    cardToken = "card-token-123abc",
    userId = "user12345",
    orderId = "SUB-2025-001",
)
freedomApi.createCardPayment(
    paymentRequest = paymentRequest
) { result: FreedomResult<PaymentResponse> ->
    when (result) {
        is FreedomResult.Success -> {
            // Payment initiation completed successfully.
        }
        is FreedomResult.Error -> {
            // Failed to initiate payment with SDK.
        }
    }
}
```
#### Confirm Card Payment
&emsp;The `confirmCardPayment` method is used to finalize a payment that was previously initiated with a saved card using [`createCardPayment`](#create-card-payment).

> **WARNING**
> This method requires that a `PaymentView` has been set using [`freedomApi.setPaymentView()`](#integrating-the-paymentview-optional) prior to calling it, as it will display a web-based form within the `PaymentView` for CVC entry and 3DS authentication.

&emsp;This method takes these parameters:

| Parameter   | Type                                            | Description                                                                                                                                        | Constraints/Notes |
|-------------|-------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------|-------------------|
| `paymentId` | `Long`                                          | Unique identifier of the payment to confirm. This paymentId is obtained from the [`createCardPayment`](#create-card-payment) method.               |                   |
| `onResult`  | `(FreedomResult<ConfirmPaymentStatus>) -> Unit` | Callback function that will be invoked with the result of the refund process. See [`ConfirmPaymentStatus`](#confirmpaymentstatus-structure) model. |                   |

&emsp;The process returns an [`FreedomResult<ConfirmPaymentStatus>`](#error-handling-and-results) object, which can be either:
- **Success**: Contains a `ConfirmPaymentStatus` object.
- **Error**: Specifies the type of error that occurred.

```kotlin
freedomApi.confirmCardPayment(paymentId = 123456L) { result: FreedomResult<ConfirmPaymentStatus> ->
    when (result) {
        is FreedomResult.Success -> {
            // Card payment successfully confirmed.
        }
        is FreedomResult.Error -> {
            // Failed to confirm payment.
        }
    }
}
```

#### Confirm Direct Payment
&emsp;The `confirmDirectPayment` method is used to finalize a payment that was previously initiated with a saved card using [`createCardPayment`](#create-card-payment).

&emsp;This method takes these parameters:

| Parameter   | Type                                       | Description                                                                                                                              | Constraints/Notes |
|-------------|--------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------|-------------------|
| `paymentId` | `Long`                                     | Unique identifier of the payment to confirm. This paymentId is obtained from the [`createCardPayment`](#create-card-payment) method.     |                   |
| `onResult`  | `(FreedomResult<PaymentResponse>) -> Unit` | Callback function that will be invoked with the result of the refund process. See [`PaymentResponse`](#paymentresponse-structure) model. |                   |

&emsp;The process returns an [`FreedomResult<PaymentResponse>`](#error-handling-and-results) object, which can be either:
- **Success**: Contains a `PaymentResponse` object.
- **Error**: Specifies the type of error that occurred.

```kotlin
freedomApi.confirmDirectPayment(paymentId = 123456L) { result: FreedomResult<PaymentResponse> ->
    when (result) {
        is FreedomResult.Success -> {
            // Direct payment successfully confirmed.
        }
        is FreedomResult.Error -> {
            // Failed to confirm direct payment.
        }
    }
}
```

---
### Google Pay Integration

&emsp;These methods facilitate integrating Google Pay into your application through the SDK. For comprehensive instructions on setting up Google Pay in your project (e.g., configuring PaymentDataRequest, handling the Google Pay sheet, and obtaining the payment token), please refer to the [official Google Pay API for Android documentation](https://developers.google.com/pay/api/android).

&emsp;Our SDK's methods bridge your Google Pay implementation with our payment gateway, handling the transaction processing.

#### Create Google Payment
&emsp;The `createGooglePayment` method is the first step in processing a payment via Google Pay using the SDK. This method initiates a Google Pay payment.

&emsp;This method takes these parameters:

| Parameter        | Type                                     | Description                                                                                                                                     | Constraints/Notes |
|------------------|------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------|-------------------|
| `paymentRequest` | `StandardPaymentRequest`                 | Essential details required to initiate a new payment. See [`StandardPaymentRequest`](#standardpaymentrequest-structure) model.                  |                   |
| `onResult`       | `(FreedomResult<GooglePayment>) -> Unit` | Callback function that will be invoked with the result of the Google payment initiation. See [`GooglePayment`](#googlepayment-structure) model. |                   |

&emsp;The process returns an [`FreedomResult<GooglePayment>`](#error-handling-and-results) object, which can be either:
- **Success**: Contains a `GooglePayment` object.
- **Error**: Specifies the type of error that occurred.

```kotlin
val paymentRequest = StandardPaymentRequest(
    amount = 123.45f,
    currency = Currency.KZT,
    description = "Google Pay Purchase",
    userId = "user12345",
    extraParams = hashMapOf("plan" to "premium_gpay")
)

freedomApi.createGooglePayment(
    paymentRequest = paymentRequest
) { result: FreedomResult<GooglePayment> ->
    when (result) {
        is FreedomResult.Success -> {
            // Google Pay payment initiated successfully.
        }
        is FreedomResult.Error -> {
            // Failed to initiate Google Payment with SDK
        }
    }
}
```

#### Confirm Google Payment
&emsp;The `confirmGooglePayment` method is the final step in processing a Google Pay transaction with the SDK. After you have successfully obtained the Google Pay token from the Google Pay API (e.g., from PaymentData in onActivityResult), you pass this token along with the `payment` received from [`createGooglePayment`](#create-google-payment) to this method to finalize the transaction.

&emsp;This method takes these parameters:

| Parameter  | Type                                       | Description                                                                                                                                                         | Constraints/Notes |
|------------|--------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------|
| `payment`  | `GooglePayment`                            | Payment previously obtained from a successful call to [`createGooglePayment`](#create-google-payment). See [`GooglePayment`](#googlepayment-structure) model.       |                   |
| `token`    | `String`                                   | Encrypted payment token received from the Google Pay API. This token contains the sensitive card data securely.                                                     |                   |
| `onResult` | `(FreedomResult<PaymentResponse>) -> Unit` | Callback function that will be invoked with the final result of the Google Pay transaction confirmation. See [`PaymentResponse`](#paymentresponse-structure) model. |                   |

&emsp;The process returns an [`FreedomResult<PaymentResponse>`](#error-handling-and-results) object, which can be either:
- **Success**: Contains a `PaymentResponse` object.
- **Error**: Specifies the type of error that occurred.

```kotlin
sdk.confirmGooglePayment(
    payment = googlePayment,
    token = googlePayToken
) { result: FreedomResult<PaymentResponse> ->
    when (result) {
        is FreedomResult.Success -> {
            // Google Pay payment confirmation successful
        }
        is FreedomResult.Error -> {
            // Failed to confirm Google Pay payment with SDK
        }
    }
}
```

---

### Error Handling and Results
&emsp;All asynchronous operations in the SDK (methods with an `onResult` callback) return their outcome encapsulated within a `FreedomResult` sealed interface.

&emsp;The `FreedomResult` interface has two primary states: `Success` for successful completion and `Error` for any failures.

#### `FreedomResult.Success<T>`
- Represents a successful completion of the SDK operation.
- `value: T`: Holds the actual result of the operation. The type `T` will vary depending on the method called (e.g., `Payment`, `List<Card>`, `ConfirmPaymentStatus`, `Unit`).

#### `FreedomResult.Error`
&emsp;The `Error` type is a sealed interface itself, providing distinct types of errors for more precise handling.

- `ValidationError`: Indicates that one or more inputs provided to the SDK method were invalid or did not meet specified constraints (e.g., `amount` out of range, `userId` format mismatch). 
    - `errors: List<ValidationErrorType>`: A list detailing all specific validation errors that occurred. See [`ValidationErrorType`](#table-validationerrortype) table.
- `PaymentInitializationFailed`: Indicates a general failure during the initial setup or preparation of a payment, before it reaches the transaction processing stage.
- `Transaction`: Represents an error encountered by the payment gateway during the transaction processing.
    - `errorCode: Int`: A numerical code representing a specific transaction error. You can find an up-to-date reference of error codes [here](https://customer.freedompay.kz/dev/error?lang=en)
    - `errorDescription: String?`: A human-readable description of the transaction error, if available.
- `NetworkError`: Represents errors related to network connectivity or API responses.
    - `Protocol`: Indicates an issue with the communication protocol or an unexpected response from the API.
        - `code: Int`: An HTTP status code.
        - `body: String?`: A human-readable description of the protocol error, if available.
    - `Connectivity`: Indicates problems related to the device's network connection.
        - `ConnectionFailed`: The network connection could not be established.
        - `ConnectionTimeout`: The network request timed out.
        - `Integrity`: An issue with the integrity of the network connection.
    - `Unknown`: A general network error occurred that does not fall into more specific categories.
- `InfrastructureError`: Represents errors related to the internal state or setup of the SDK.
    - `SdkNotConfigured`: The SDK methods were called before `freedomApi.setConfiguration()` was successfully invoked.
    - `SdkCleared`: The SDK methods were called after the SDK was cleared.
    - `ParsingError`: An error occurred while parsing data (e.g., response from the server could not be deserialized).
    - `WebView`: Errors specifically related to the internal `WebView` component used for displaying payment pages/frames.
        - `PaymentViewIsNotInitialized`: A method requiring `PaymentView` was called, but `freedomApi.setPaymentView()` has not been called.
        - `Failed`: An error occurred during the payment process.

#### Table `ValidationErrorType`

| Enum Constant            | Description                                                                |
|--------------------------|----------------------------------------------------------------------------|
| `INVALID_MERCHANT_ID`    | Provided merchant ID is invalid or missing.                                |
| `INVALID_SECRET_KEY`     | Provided secret key is invalid or missing.                                 |
| `INVALID_PAYMENT_AMOUNT` | Payment amount is outside the allowed range (`0.01` - `99999999.00`).      |
| `INVALID_ORDER_ID`       | Provided order ID does not meet the specified length constraints.          |
| `INVALID_USER_ID`        | Provided user ID does not meet the specified format or length constraints. |
| `INVALID_CARD_TOKEN`     | Provided card token does not meet the specified length constraints.        |

---

### Data Structures
&emsp;This section provides detailed descriptions of the data classes and enums used throughout the SDK, particularly in the results of various method calls.

#### `StandardPaymentRequest` Structure
&emsp;The `StandardPaymentRequest` data class encapsulates the essential details required to initiate a new payment transaction.

| Parameter     | Type                               | Description                                                                                | Constraints/Notes                                                                                    |
|---------------|------------------------------------|--------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| `amount`      | `Float`                            | Total amount of the payment.                                                               | Must be between `0.01` and `99999999.00`.                                                            |
| `currency`    | `Currency?`                        | Currency of the payment. See [`Currency`](#currency-structure) enum for available options. | Optional. Defaults to null.                                                                          |
| `description` | `String`                           | Description of the payment.                                                                |                                                                                                      |
| `userId`      | `String?`                          | Identifier for the user making the payment.                                                | Optional. Defaults to null. Must contain 1-50 characters. Also, must match regex `^[a-zA-Z0-9_-]+$`. |
| `orderId`     | `String?`                          | Unique identifier for this payment order within your system.                               | Optional. Defaults to null. Must contain 1-50 characters.                                            |
| `extraParams` | `HashMap<String, String>?`         | Optional map of additional key-value pairs to pass custom data with the payment.           | Optional. Defaults to null.                                                                          |

#### `TokenizedPaymentRequest` Structure
&emsp;The `TokenizedPaymentRequest` data class is used to create payments with a previously saved and tokenized card, requiring the card's unique token along with standard payment details.

| Parameter     | Type                                                 | Description                                                                                                                                                                    | Constraints/Notes                                                        |
|---------------|------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| `amount`      | `Float`                                              | Total amount of the payment to be charged to the saved card..                                                                                                                  | Must be between `0.01` and `99999999.00`.                                |
| `currency`    | `Currency?`                                          | Currency of the payment. See [`Currency`](#currency-structure) enum for available options.                                                                                     | Optional. Defaults to null.                                              |
| `description` | `String`                                             | Description of the payment.                                                                                                                                                    |                                                                          |
| `cardToken`   | `String`                                             | Unique token representing the saved card that will be used for this payment. This token is obtained from [`addNewCard`](#add-new-card) or [`getAddedCards`](#get-added-cards). | Must contain 1 or more characters.                                       |
| `userId`      | `String`                                             | Identifier for the user making the payment.                                                                                                                                    | Must contain 1-50 characters. Also, must match regex `^[a-zA-Z0-9_-]+$`. |
| `orderId`     | `String`                                             | Unique identifier for this payment order within your system.                                                                                                                   | Must contain 1-50 characters.                                            |
| `extraParams` | `HashMap<String, String>?`                           | Optional map of additional key-value pairs to pass custom data with the payment.                                                                                               | Optional. Defaults to null.                                              |

#### `PaymentResponse` Structure
&emsp;The `PaymentResponse` data class represents a successful payment transaction.

| Property     | Type      | Description                                     |
|--------------|-----------|-------------------------------------------------|
| `status`     | `String`  | Status of the operation.                        |
| `paymentId`  | `Long`    | Unique identifier for this payment.             |
| `merchantId` | `String`  | ID of the merchant associated with the payment. |
| `orderId`    | `String?` | Order ID provided during payment creation.      |

#### `Status` Structure
&emsp;Provides comprehensive details about the current state of a payment.

| Property          | Type                    | Description                                                             |
|-------------------|-------------------------|-------------------------------------------------------------------------|
| `status`          | `String`                | Status of the operation.                                                |
| `paymentId`       | `Long`                  | Unique identifier for this payment.                                     |
| `orderId`         | `String?`               | Order ID provided during payment creation.                              |
| `currency`        | `String`                | Currency code of the payment.                                           |
| `amount`          | `Float`                 | Original amount of the payment.                                         |
| `canReject`       | `Boolean?`              | Indicates if the payment can still be cancelled.                        |
| `paymentMethod`   | `String?`               | Method used for payment.                                                |
| `paymentStatus`   | `String?`               | Current status of the payment.                                          |
| `clearingAmount`  | `Float?`                | Total amount that has been cleared (captured) for this payment.         |
| `revokedAmount`   | `Float?`                | Total amount that has been cancelled for this payment.                  |
| `refundAmount`    | `Float?`                | Total amount that has been refunded.                                    |
| `cardName`        | `String?`               | Name on the card used for the payment.                                  |
| `cardPan`         | `String?`               | Masked Primary Account Number (PAN) of the card.                        |
| `revokedPayments` | `List<RevokedPayment>?` | List of individual cancelled transactions associated with this payment. |
| `refundPayments`  | `List<RefundPayment>?`  | List of individual refund transactions associated with this payment.    |
| `reference`       | `Long?`                 | System-generated reference number for the payment.                      |
| `captured`        | `Boolean?`              | Indicates if the funds for the payment have been captured.              |
| `createDate`      | `String`                | Date and time when the payment was created.                             |
| `authCode`        | `Int?`                  | Authorization code for the payment.                                     |

#### `RevokedPayment` Structure
&emsp;Details of an individual cancelled transaction.

| Property        | Type      | Description                      |
|-----------------|-----------|----------------------------------|
| `paymentId`     | `Long?`   | ID of the cancelled payment.     |
| `paymentStatus` | `String?` | Status of the cancelled payment. |

#### `RefundPayment` Structure
&emsp;Details of an individual refund transaction.

| Property        | Type      | Description                                       |
|-----------------|-----------|---------------------------------------------------|
| `paymentId`     | `Long?`   | ID of the refund payment.                         |
| `paymentStatus` | `String?` | Status of the refund payment.                     |
| `amount`        | `Float?`  | Amount that was refunded in this transaction.     |
| `createDate`    | `String?` | Date and time of the refund.                      |
| `reference`     | `Long?`   | System-generated reference number for the refund. |

#### `ClearingStatus` Structure
&emsp;Represents the status of a clearing operation.

| Type                         | Description                                                                                               |
|------------------------------|-----------------------------------------------------------------------------------------------------------|
| `Success(val amount: Float)` | Indicates that the clearing operation was successful. `amount`: The amount that was successfully cleared. |
| `ExceedsPaymentAmount`       | Indicates that the requested clearing amount exceeded the originally authorized payment amount.           |
| `Failed`                     | Indicates that the clearing operation failed.                                                             |

#### `Card` Structure
&emsp;Represents a single tokenized payment card associated with a user.

| Property             | Type      | Description                                            |
|----------------------|-----------|--------------------------------------------------------|
| `status`             | `String?` | Status of the operation.                               |
| `merchantId`         | `String?` | ID of the merchant.                                    |
| `recurringProfileId` | `String?` | ID assigned to a user's recurring payment profile      |
| `cardToken`          | `String?` | Token used to reference this card for future payments. |
| `cardHash`           | `String?` | Masked Primary Account Number (PAN) of the card.       |
| `createdAt`          | `String?` | Date and time when the card was added.                 |

#### `RemovedCard` Structure
&emsp;Represents the outcome of an attempt to remove a stored card.

| Property     | Type      | Description                                      |
|--------------|-----------|--------------------------------------------------|
| `status`     | `String?` | Status of the operation.                         |
| `merchantId` | `String?` | ID of the merchant.                              |
| `cardHash`   | `String?` | Masked Primary Account Number (PAN) of the card. |
| `deletedAt`  | `String?` | Date and time when the card was deleted.         |

#### `ConfirmPaymentStatus` Structure
&emsp;Represents the basic outcome of confirming a card payment.

| Type      | Description                                                                      |
|-----------|----------------------------------------------------------------------------------|
| `Success` | Indicates that the card payment confirmation process was completed successfully. |
| `Failure` | Indicates that the card payment confirmation process failed.                     |

#### `GooglePayment` Structure
&emsp;The response received after initiating a Google Pay transaction.

| Property    | Type     | Description                                                 |
|-------------|----------|-------------------------------------------------------------|
| `paymentId` | `String` | Unique identifier for the initiated Google Pay transaction. |

#### `Currency` Structure
&emsp;The `Currency` enum defines all supported currency codes that can be used when specifying payment amounts.
```kotlin
enum class Currency(val value: String) {
    KZT("KZT"),
    RUB("RUB"),
    USD("USD"),
    UZS("UZS"),
    KGS("KGS"),
    EUR("EUR"),
    HKD("HKD"),
    GBP("GBP"),
    AED("AED"),
    CNY("CNY"),
    KRW("KRW"),
    INR("INR"),
    THB("THB"),
    UAH("UAH"),
    AMD("AMD"),
    BYN("BYN"),
    PLN("PLN"),
    CZK("CZK"),
    AZN("AZN"),
    GEL("GEL"),
    TJS("TJS"),
    CAD("CAD"),
    MDL("MDL"),
    TRY("TRY")
}
```

---
### Support
- **If you have questions or need help, feel free to reach out!** 👋
- **Email**: support@freedompay.kz

