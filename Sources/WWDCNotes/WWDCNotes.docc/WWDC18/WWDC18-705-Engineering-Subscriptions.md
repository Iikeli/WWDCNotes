# Engineering Subscriptions

Learn the best practices for architecting your subscription infrastructure using StoreKit and server-side logic. Find out about simple engineering techniques to keep your subscribers longer, and how to utilize new tools and APIs to give your subscribers the best experience.

@Metadata {
   @TitleHeading("WWDC18")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc18/705", purpose: link, label: "Watch Video (44 min)")

   @Contributors {
      @GitHubUser(antonio081014)
      @GitHubUser(javierdemartin)
   }
}



## 1. Device and Server Architecture

### 1. Receive Transaction

Purchases or suscriptions are handled using the [`StoreKit`](https://developer.apple.com/documentation/storekit#) framework. Every time a transaction occurs, `StoreKit` informs your app of these transactions via [`SKPaymentTransactionObserver`](https://developer.apple.com/documentation/storekit/skpaymenttransactionobserver#). 

This transaction observer is the central piece of IAPs (In-App Purchases). It's a protocol and you can set it to any object. It's important to add a `TransactionObserver` to the default payment queue as early as possible in the application lifecycle so you will receive transactions as they occur in the background via the [`paymentQueue(_:updatedTransactions:)`](https://developer.apple.com/documentation/storekit/skpaymenttransactionobserver/1506107-paymentqueue#) callback in the TransactionObserver.

```swift
class AppDelegate: ..., SKPaymentTransactionObserver {

	// Your app wil be informed of a new set of transactions to process.
	func paymentQueue(_ queue: SKPaymentQueue, updatedTransactions transactions: [SKPaymentTransaction]) {

		for transaction in transactions {
			switch transaction.transactionState {
				case .purchased:
					// Validate the purchase, ready for verification and unlocking
			}
		}
	}
}
```

Transactions can come in various [states](https://developer.apple.com/documentation/storekit/skpaymenttransactionstate) and it's your task to decide what to do on each case.

### 2. Verify Authenticity

A transaction can be deemed valid if money has changed hands meaning that the user has completed a valid purchase operation. These operations will be reflected on the [App Store Receipt](https://developer.apple.com/documentation/appstorereceipts).

#### App Store Receipt
    
It's a trusted record of App and In-App purchases, stored on device and issued by the App Store. One user with multiple devices has receipts that look slightly different. Contains details of the initial app download and any in-app purchases that have occurred for this app. 

#### Receipt Validation

First step is to verify that the document, App Store Receipt, is authentic and it was issued by Apple. This can be done in [two ways](https://developer.apple.com/documentation/storekit/in-app_purchase/choosing_a_receipt_validation_technique):

1. **On-device validation**: Apply local rules to check the certificates used are valid and the receipt is valid.
2. **Server-to-server**: Take the binary encoded receipt data, send it to your own server and relay it to the App Store for processing. This delegates all of the checks to the App Store.
    
**Don't** use online validation directly from the device as the receipt can be tampered for malicious purposes.

Both methods return access to the contents of the receipt, any transactions that have occurred for this particular user. **Apple encourages to adopt Server-to-server validation** when it comes to maintaining an auto renewable suscription sate.

|                                          | On-device | Server-to-Server |
|------------------------------------------|-----------|------------------|
| Validate authenticity of receipt         | x         | x                |
| Receipts includes renewal transactions   | x         | x                |
| Additional user subscription information |           | x                |
| Always "on" to handle renewals           |           | x                |
| Not susceptible to device "clock change" |           | x                |
| No cryptography needed                   |           | x                |

> Capabilities comparison between the two receipt validation methods.

#### Transaction processing using Server-to-Server Validation

In the transaction observer you can access the binary receipt data using the [`appStoreReceiptURL`](https://developer.apple.com/documentation/foundation/bundle/1407276-appstorereceipturl#) on the main bundle. This URL contains the binary data that you can encode it using Base64 and send it to your server for processing stated in the previous methods.

```swift
switch transaction.transactionState {
	case .purchased:
		if let appStoreReceiptURL = Bundle.main.appStore.ReceiptURL,
			FileManager.default.fileExists(atPath: appStoreReceiptURL.path) {

			let rawReceiptData = Data(contentsOf: appStoreReceiptURL)
			let receiptData = rawReceiptData.base64EncodedString(options: ...)

			// Send up receiptString to the server to validate
			currentUser.processTransaction(receiptData) { isValid in

			}
		}
}
```

**How to send the encoded data to your server?** 

1. Enable an endpoint for this operation such as `/processTransaction` where you will send all of the relevant data for this operation.
2. Add as a payload the Base64 encoded receipt data, `{ receiptData: "e2fFt...xe4aW=" }`.
3. Once the data has reached your server you can securely establish a secure connection to the App Store's own `/verifyReceipt` endpoint. You can include a password field that is a shared secret between your app and the AppStore.

```
{ receipt-data: "e2fFt...xe4aW=",
	password: "a41hd732gav" } 
```

4. Apple's `/verifyReceipt` will respond with a JSON payload. All of the available fields in the response are detailed [here](https://developer.apple.com/library/archive/releasenotes/General/ValidateAppStoreReceipt/Chapters/ReceiptFields.html). To verify if the transaction is authentic check the `status` field. This indicates if Apple has actually issued this document in the first place. Afterwards, check the contents of the receipt, this is the decoded version of the binary data sent to `/verifyReceipt` endpoint. Check if the bundle ID matches the one in your app. The `in_app` array contains a list of transactions for this user. Verify if the `product_id` associated with this receipt is the one associated with your app. If all these match, this receipt entitles this user to your suscription product and you can proceed to update this user's subscription status.

```
{ status: 0,
	receipt: {
		bundle_id: "com.your.app",
		in_app: [{
			transaction_id: "1234567890",
			product_id: "com.your.product.id",
			original_transaction_id: "1133557799",
			expires_date: "2018-07-08",
			...
		}]
	}
}
```

### 3. Update Subscription Status

To keep track of each subscription ID and status Apple recommends to create some sort of database on your server to keep track of each user's operations. Determine depending on your needs which fields you need.

| userID   | originalTransactionId | latestExpiresDate |
|----------|-----------------------|-------------------|
| 90000001 | 11133557799           | 2018-08-08        |
| 90000002 | 2244668800            | 2018-02-02        |
| 90000003 | 0099887766            | 2018-04-20        |

Every suscription period starts with a transaction and ends with an expiry date. By inspecting the response from `/verifyReceipt` we can identify each of these expiry dates for each transaction. The `latest_expires_date` might be the source of truth you need to know if this user is a subscriber. Use it along `original_transaction_id`  for relating together mulitple transaction receipts for the same individual customer's subscription.

Each user who purchases a suscription will be assigned a **unique `original_transaction_id` which is essentially the user's subscription id**. It shows up in all subsequent renewal transactions.

Once the transaction has been verified call [`finishTransaction`](https://developer.apple.com/documentation/storekit/skpaymentqueue/1506003-finishtransaction#). This will clear it out of your transaction payment queue. If it's not cleared out it might appear next time the app launches for processing. **Finish every transaction that begins in StoreKit**.

```swift
switch transaction.transactionState {
	case .purchased:
		if let appStoreReceiptURL = Bundle.main.appStore.ReceiptURL,
			FileManager.default.fileExists(atPath: appStoreReceiptURL.path) {


			let rawReceiptData = Data(contentsOf: appStoreReceiptURL)
			let receiptData = rawReceiptData.base64EncodedString(options: ...)


			// Send up receiptString to server
			currentUser.processTransaction(receiptData) { isValid in
				if isValid {
					queue.finishTransaction(transaction)
				}
			}
		}
}
```
  
How to check if a user has active subscription on your side?

1. Filter transactions by `original_transaction_id`
2. Find the transaction with latest `expires_date`
3. If the latest one is in the past that means the user does not have a subscription active. If the latest date is in the future a subscription is active.

#### Status Polling

Discover new transactions directly from your server.
  
  - Save latest version of encoded receipt data on your serer(latestReceiptData, exclude-old-transactions: true)
  - Treat receipt data like a token
  - `/verifyReceipt` response also includes new transactions
  - Located in the `latest_receipt_info` field
  - Unlock new subscription periods without waiting for app to launch
  
  New transactions will still appear in StoreKit on next app launch
  - Must verify and finish these transactions
  - Even if your server already knows about them
  - Opportunity to update latest receipt data on server

#### Reacting to Billing Issues

1. Observe no renewal transaction appears, the subscription has lapsed. 
2. Direct user to amend their billing details
3. Unblock user immediately when transaction occurs

#### Server-to-Server Notifications

Receive [HTTPS POST requests](https://developer.apple.com/documentation/storekit/in-app_purchase/subscriptions_and_offers/enabling_app_store_server_notifications) to an endpoint on your own server by entering a URL in App Store Connect. The App Store will begin to send your server requests for status change events. Includes `latest_transaction_info` for the transaction in question. It's your chore now to update that subscription status so the user receives the latest information and billing status.

You need to make sure your server is ATS compliant in order to receive these.

## 2. In-App Experience

Tips to enhance user's experience with in-app purchases.

1. **Offer in-app purchases before an account creation**. Less overhead for the user, they only need to open the app and buy the subscription to get access to the content your're after. If you want to associate multiple devices do so by using the `original_transaction_id` field. This will be anonymous for your users. If you want to register the user's email add a field in your database that will link that transaction field to the user's email.
2. **Sell your in-app purchases with introductory pricing**. You need to check if a user is elegible for an introductory price so you know what price to render to your user. Do it by checking transactions to see if discount has been used. The receipt can help with this using two extra fields: `is_trial_period` and `is_in_intro_offer_period`. If either of those are `true` it's an indication that an introductory offer or a free trial was used for this particular transaction. If it was you should keep track of the product ID in question against this current user under a field called `consumedProductDiscounts`.
3. **Upgrade/Downgrade Subscriptions**: Offer users upgrades and downgrades between subscription tiers right in your app's UI. Treat it just like selling an initial subscription. If the **subscription you're selling** the user is **part of the same subscription group** so it's a different tier than the one the user has already subscribed just create an `SKPayment` just like you would do if selling an initial subscription. StoreKit automatically handles the upgrade/downgrade for you so you don't have to worry if the user has subscribed twice. 
4. **Subscription Management**, you can provide a [link](https://apps.apple.com/account/subscriptions) to the App Store's subscription management pane instead of creating your own interface.

### How to keep track of discounts and introductory offers

It's useful to know which price to render when offering an user a price for a subscription.

```swift
func fetchConsumedGroupDiscounts() {
    // 1. Take the consumed product discounts saved against the current user
    let consumedProducts = currentUser.consumedProductDiscounts
    // 2. Execute a SKProductsRequest with them
    let productsRequest = SKProductsRequest(productIdentifiers: Set(consumedProducts))
    productsRequest.delegate = self
    productsRequest.start()
}

func productsRequest(_ request: SKProductsRequest, didReceive response: SKProductsResponse) {
    for product in response.products {
        // 3. The response from SKProductRequest includes the subscription group identifier
        // you know which subscription group this particular product is from.
        // With this subscription group identifier you can keep track of that in a
        // Set of consumed group discounts for this particular user.
        // Now you know which subscription groups this user has used offers for.
        currentUser.consumedGroupDiscounts.append(product.subscriptionGroupIdentifier)
    } 
}
```

When it's time to render the product price tag for a given product it becames a simple check against that `Set`.

```swift
let price: NSDecimalNumber
let priceLocale: Locale
// 4. Check if this user's list of consumed group discounts contains the group identifier
// for the one you want to sell them.
if currentUser.consumedGroupDiscounts.contains(productA.subscriptionGroupIdentifier) {
    // 5.1. The user has used an introductory offer before. Rencer the normal price string
    price = productA.price
    priceLocale = productA.priceLocale
} else {
    // 5.2. The user has not used an introductory offer before. They are elegible for that discounted offer.
    price = productA.introductoryPrice.price
    priceLocale = productA.introductoryPrice.priceLocale
}

// 6. Present the correctly 
let formatter = NumberFormatter()
formatter.numberStyle = .currency
formatter.locale = priceLocale
let formattedString = formatter.string(from: price)
```

## 3. Reducing Subscriber Churn

Problems that might appear when dealing  with subscriptions:

1. [**Involuntary churn**](https://developer.apple.com/documentation/storekit/in-app_purchase/subscriptions_and_offers/reducing_involuntary_subscriber_churn) is the loss of a subscriber due to a failed payment or billing issue on the plaftorm. If the renewal failed and you have set up server-to-server notifications you receive a `DID_FAIL_TO_RENEW` notification. You can give free days of unpaid service, grace period, to not have to reacquire those users until the billing issues have been solved. The field `is_in_billing_retry_period` reflects this behavior. If billing issues appear you can [redirect](https://apps.apple/com/account/billing) users to edit their billing information. When a retry attempt is successful the date of retry or recovery becomes the new subscription aniversary date moving forward. This will be reflected in the JSON response.
2. **Voluntary churn** is the loss of a subscriber due to customer choice, either via cancelation or refund requests. Implement status polling and/or offer attractive alternative subscriptions to potentially save that user. The `auto_renew_status = 1` field shows that that subscriber will return in the subsequent subscription period. 
3. **Winback**.

Solution:

1. Leverage subscription specific [receipt fields](https://developer.apple.com/library/archive/releasenotes/General/ValidateAppStoreReceipt/Chapters/ReceiptFields.html) 
2. Implement status polling to know when your users will voluntarily churn.
3. Implement customer messaging
4. Present contextual subscription offers to hopefully win them back or churning in the first place.

You can check the reason why a user's subscription has expired by exploring the field `expiration_intent`.

## 4. Analytics and Reporting

[App Store Connect](http://appstoreconnect.apple.com) contains a huge amount of useful information. 

## Summary

- Server side state management offers more flexibility
- Use notifications from the App Store
- Offer introductory pricing
- Reduce churn with simple messaging
- Win back users with alternative subscription options
- New reporting tools in App Store Connect
