# Migration Guide

## Migrating from versions < 1.2.0

### Google Pay
- `FreedomAPI.createGooglePayment()` and `FreedomAPI.confirmGooglePayment()` have been removed. Use `FreedomAPI.payWithGooglePay()` instead.
  - Google Pay payments now take a single SDK call. You no longer create a payment before showing the Google Pay sheet. Launch Google Pay directly, then pass the received token to `payWithGooglePay()`.
  - `payWithGooglePay()` takes the same `StandardPaymentRequest` you previously passed to `createGooglePayment()`, together with the Google Pay token.
  - The result type is unchanged: `FreedomResult<PaymentResponse>`.
  ```kotlin
  // before
  googlePayButton.setOnClickListener {
      freedomApi.createGooglePayment(paymentRequest) { result ->
          when (result) {
              is FreedomResult.Success -> {
                  googlePayment = result.value
                  AutoResolveHelper.resolveTask(
                      googlePaymentsClient.loadPaymentData(createPaymentDataRequest()),
                      this,
                      REQUEST_CODE
                  )
              }
              is FreedomResult.Error -> { /* handle error */ }
          }
      }
  }

  // in onActivityResult
  freedomApi.confirmGooglePayment(
      payment = googlePayment,
      token = googlePayToken
  ) { result: FreedomResult<PaymentResponse> -> /* handle result */ }

  // after
  googlePayButton.setOnClickListener {
      AutoResolveHelper.resolveTask(
          googlePaymentsClient.loadPaymentData(createPaymentDataRequest()),
          this,
          REQUEST_CODE
      )
  }

  // in onActivityResult
  freedomApi.payWithGooglePay(
      paymentRequest = paymentRequest,
      token = googlePayToken
  ) { result: FreedomResult<PaymentResponse> -> /* handle result */ }
  ```
- `GooglePayment` has been removed. It was only used to pass the payment from `createGooglePayment()` to `confirmGooglePayment()` and is no longer needed.
- `payWithGooglePay()` requires a `PaymentView`. Call `freedomApi.setPaymentView()` before paying. If the bank requires 3-D Secure, the SDK shows the bank's page in the `PaymentView` and hides it once the payment gets a final status. Without a `PaymentView`, the call returns `PaymentViewIsNotInitialized`.
  ```kotlin
  // after
  freedomApi.setPaymentView(paymentView)
  ```
- To let the user cancel 3-D Secure (for example, on a Back press), hide the `PaymentView`. The SDK then returns the latest known payment status.
  ```kotlin
  // after
  paymentView.visibility = View.GONE
  ```
- `payWithGooglePay()` accepts an optional `force3DS` parameter (defaults to `false`). Set it to `true` to require 3-D Secure for the payment.
  ```kotlin
  // after
  freedomApi.payWithGooglePay(
      paymentRequest = paymentRequest,
      token = googlePayToken,
      force3DS = true
  ) { result -> /* handle result */ }
  ```
- A declined payment is now returned as `FreedomResult.Success` with `PaymentResponse.status` set to `Status.Error`. Check `status` instead of relying on `FreedomResult.Error` alone.
  ```kotlin
  // after
  when (result) {
      is FreedomResult.Success -> when (val status = result.value.status) {
          PaymentResponse.Status.Success -> { /* payment completed */ }
          is PaymentResponse.Status.Error -> { /* declined: status.code, status.description */ }
          else -> { /* not final yet: check later with getPaymentStatus() */ }
      }
      is FreedomResult.Error -> { /* payment was not made */ }
  }
  ```
- If `payWithGooglePay()` returns `FreedomResult.Error.NetworkError`, the payment may still have been made. Check its status on your backend by `orderId` before letting the user pay again. We recommend always setting `orderId` in `StandardPaymentRequest`.

See [Paying with Google Pay](README.md#5-paying-with-google-pay) for the full integration.
