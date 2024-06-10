# What’s new in Wallet and Apple Pay

Discover the latest updates to Wallet & Apple Pay. We'll show you how to support Orders in Wallet for your apps and websites and securely validate someone's age and identity with the Identity Verification API. We'll also explore PassKit support for SwiftUI, and discuss how you how you can improve your Apple Pay experience with Automatic Payments.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/10041", purpose: link, label: "Watch Video (36 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Updates

### Tap to pay

- from iOS 15.4
- provides contactless payments support
- includes Apple Pay, contactless credit and debit cards, and other digital wallets
- the transaction is completed through a tap to the iPhone
- no need for additional hardware or payment terminals

### New Apple Pay UX in macOS Ventura

- written in SwiftUI (shared codebase with iOS)

### New SwiftUI APIs

- [`AddPassToWalletButton`][AddPassToWalletButton]
  - use `.frame(...)` for different sizes 
  - supports styling via [`AddPassToWalletButtonStyle`][AddPassToWalletButtonStyle]

- [`PayWithApplePayButton`][PayWithApplePayButton]
  - use `.frame(...)` for different sizes 
  - supports styling via [`PayWithApplePayButtonStyle`][PayWithApplePayButtonStyle]

## Multi-merchant payments

Useful for things like online marketplaces, travel bookings, and ticketing services

```swift
// Create a payment request
let paymentRequest = PKPaymentRequest()
// ...

// Set total amount
paymentRequest.paymentSummaryItems = [
  PKPaymentSummaryItem(label: "Total", amount: 500)
]

// Create a multi token context for each additional merchant in the payment
let multiTokenContexts = [
  PKPaymentTokenContext(
    merchantIdentifier: "com.example.air-travel",
    externalIdentifier: "com.example.air-travel",
    merchantName: "Air Travel",
    merchantDomain: "air-travel.example.com",
    amount: 150
  ),
  PKPaymentTokenContext(
    merchantIdentifier: "com.example.hotel",
    externalIdentifier: "com.example.hotel",
    merchantName: "Hotel",
    merchantDomain: "hotel.example.com",
    amount: 300
  ),
  PKPaymentTokenContext(
    merchantIdentifier: "com.example.car-rental",
    externalIdentifier: "com.example.car-rental",
    merchantName: "Car Rental",
    merchantDomain: "car-rental.example.com",
    amount: 50
  )
]
paymentRequest.multiTokenContexts = multiTokenContexts
```

## Automatic payments (including Subscriptions)

- two types support:
  - recurring payments
  - automatic reload payments

### Recurring payments

- subscriptions, installments, or recurring billing
- fixed or variable amount charged on a regular schedule
- end on a specific date or until canceled
- trial or introductory period

```swift
// Specify the amount and billing periods
let regularBilling = PKRecurringPaymentSummaryItem(label: "Membership", amount: 20)

let trialBilling = PKRecurringPaymentSummaryItem(label: "Trial Membership", amount: 10)

let trialEndDate = Calendar.current.date(byAdding: .month, value: 1, to: Date.now)
trialBilling.endDate = trialEndDate
regularBilling.startDate = trialEndDate

// Create a recurring payment request
let recurringPaymentRequest = PKRecurringPaymentRequest(
  paymentDescription: "Book Club Membership",
  regularBilling: regularBilling,
  managementURL: URL(string: "https://www.example.com/managementURL")!
)
recurringPaymentRequest.trialBilling = trialBilling

recurringPaymentRequest.billingAgreement = """
50% off for the first month. You will be charged $20 every month after that until you cancel. \ You may cancel at any time to avoid future charges. To cancel, go to your Account and click \ Cancel Membership.
"""

recurringPaymentRequest.tokenNotificationURL = URL(
  string: "https://www.example.com/tokenNotificationURL"
)!

// Update the payment request
let paymentRequest = PKPaymentRequest()
// ...
paymentRequest.recurringPaymentRequest = recurringPaymentRequest

// Include in the summary items
let total = PKRecurringPaymentSummaryItem(label: "Book Club", amount: 10)
total.endDate = trialEndDate
paymentRequest.paymentSummaryItems = [trialBilling, regularBilling, total]
```

### Automatic reload payments

- e.g., store card balance top-ups, pre-paid accounts top-ups
- Fixed top-up amount payment
- Charged whenever account balance drops below a set threshold amount

```swift
// Specify the reload amount and threshold
let automaticReloadBilling = PKAutomaticReloadPaymentSummaryItem(
  label: "Coffee Shop Reload",
  amount: 25
)
reloadItem.thresholdAmount = 5

// Create an automatic reload payment request
let automaticReloadPaymentRequest = PKAutomaticReloadPaymentRequest(
  paymentDescription: "Coffee Shop",
  automaticReloadBilling: automaticReloadBilling,
  managementURL: URL(string: "https://www.example.com/managementURL")!
)

automaticReloadPaymentRequest.billingAgreement = """
Coffee Shop will add $25.00 to your card immediately, and will automatically reload your \
card with $25.00 whenever the balance falls below $5.00. You may cancel at any time to avoid \ future charges. To cancel, go to your Account and click Cancel Reload.
"""

automaticReloadPaymentRequest.tokenNotificationURL = URL(
  string: "https://www.example.com/tokenNotificationURL"
)!

// Update the payment request
let paymentRequest = PKPaymentRequest()
// ...
paymentRequest.automaticReloadPaymentRequest = automaticReloadPaymentRequest

// Include in the summary items
let total = PKAutomaticReloadPaymentSummaryItem(
  label: "Coffee Shop",
  amount: 25
)
total.thresholdAmount = 5
paymentRequest.paymentSummaryItems = [total]
```

## Identity verification

Refer to [Verifying Wallet identity requests][a1] and [Requesting identity data from a Wallet pass][a2].

[AddPassToWalletButton]: https://developer.apple.com/documentation/passkit/addpasstowalletbutton
[PayWithApplePayButton]: https://developer.apple.com/documentation/passkit/paywithapplepaybutton
[AddPassToWalletButtonStyle]: https://developer.apple.com/documentation/passkit/AddPassToWalletButtonStyle
[PayWithApplePayButtonStyle]: https://developer.apple.com/documentation/passkit/PayWithApplePayButtonStyle
[a1]: https://developer.apple.com/documentation/passkit/wallet/verifying_wallet_identity_requests
[a2]: https://developer.apple.com/documentation/passkit/wallet/requesting_identity_data_from_a_wallet_pass
