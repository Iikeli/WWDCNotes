# Meet StoreKit 2

StoreKit 2 delivers powerful, Swift-native APIs for in-app purchases and auto-renewable subscriptions. Learn how you can easily implement in-app purchases and subscriptions, and discover APIs for retrieving product information, handling transactions, determining product entitlements and customer status, as well as comprehensive testing support in Xcode.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10114", purpose: link, label: "Watch Video (37 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}




> [Session sample code][samplecode]

## StoreKit 2

- brand new set of modern and flexible Swift APIs for working with In-App Purchase across iOS, macOS, tvOS, and watchOS
- updates to the in-app purchase transactions, to make it easier to manage

Five main areas:

- Products
- Purchases
- Transaction info
- Transaction history
- Subscription status

## Products & Purchases

- New [`Product`][product] struct
  - contains additional data, such as the product type ([`Product.ProductType`][Product.ProductType]) and extended subscription information ([`Product.SubscriptionInfo`][Product.SubscriptionInfo])
  - future-compatible with new features via [`BackingValue`][BackingValue], which allows you to retrieve data contained in the product even on SDKs and devices running operating systems that have older versions of StoreKit 2
  - uses `async`/`await` for requests and purchases

- purchase is a `Product` instance method, [`purchase(options:)`][purchase(options:)]
  - the options parameter lets you specify things like quantity, promotional offer, app account token (New)
  - the [App account token][appAccountToken(_:)] is a way for you to keep track of which of your app's user accounts began and completed a transaction.
    - it's an opaque token (UUID format)
    - created by you
    - linked to the user in-app account
    - stored in transaction forever

## Transaction info

- individually signed object for every transaction
- in-app purchase transaction info will now be provided in [JSON Web signature][jws]
- StoreKit 2 does transaction verification for you, but you can also add your verification on top

## Transaction history

- new set of APIs for querying completed transactions in the user's transaction history
- you can access all of the user's past transactions with a single API call
- you can also access the latest transaction for a product
- get what products the user has paid for access via [`currentEntitlements`][ce]
  - contains all of the non-consumables in the user's transaction history
  - contains all of the subscription transactions that are currently active
  - any transactions that have been revoked are **not** included
  - no consumable transactions

- all transactions are available immediately upon app download
- transactions automatically update on every device
- real-time updates
- thanks to transaction history, users no longer need to restore completed transactions when your app is reinstalled
- you still need to provide a UI to restore purchases
- all new StoreKit 2 transactions are available to the old StoreKit API
- all old StoreKit transactions are available to the new StoreKit 2 API

## Subscription status

A [subscription][sub] status has three parts:

1. latest transaction
  - give you access to the last transaction that occurred for this subscription

2. renewal state
  - enumeration that tells you the current state of the subscription 
  - base you app logic off this renewal state

3. renewal info
  - contains all details about a user's subscription (e.g. whether auto-renew is on/off, product id of the renewal, expiration reason)
  - signed using JWS
  - StoreKit 2 will automatically validate the renewal info for you

## Signature validation

> More details in [RFC7515][RFC7515]

JSON Web Signature is comprised of three parts:

1. header
  - contains metadata about the object, such as which algorithm is used for signing and where to find the certificate used to validate the signature. 
  - StoreKit 2 currently uses an `ECDSA` algorithm (supported natively in CryptoKit)
  - For the certificate, StoreKit 2 uses the `x5c` header 
  - the entire certificate chain is included in the JWS data, no internet connection

2. payload
  - main transaction information such as transaction ID, product ID, purchase date

3. signature
  - generated using both the header and the payload

## Demo

The session has two demos touching all the flows:

- the first demo focuses on getting products, purchasing, and listening to transitions. 
- the second demo focuses on transaction history and subscription statuses
- the final code can be found [here][samplecode]

[product]: https://developer.apple.com/documentation/storekit/product
[Product.ProductType]: https://developer.apple.com/documentation/storekit/product/producttype
[Product.SubscriptionInfo]: https://developer.apple.com/documentation/storekit/product/subscriptioninfo
[BackingValue]: https://developer.apple.com/documentation/storekit/backingvalue
[purchase(options:)]: https://developer.apple.com/documentation/storekit/product/3791971-purchase
[appAccountToken(_:)]: https://developer.apple.com/documentation/storekit/product/purchaseoption/3749440-appaccounttoken
[jws]: https://www.google.com/search?client=safari&rls=en&q=JSON+Web+signature&ie=UTF-8&oe=UTF-8
[samplecode]: https://developer.apple.com/documentation/storekit/in-app_purchase/implementing_a_store_in_your_app_using_the_storekit_api
[ce]: https://developer.apple.com/documentation/storekit/transaction/3792059-currententitlements
[sub]: https://developer.apple.com/documentation/storekit/product/3749590-subscription
[RFC7515]: https://datatracker.ietf.org/doc/html/rfc7515
