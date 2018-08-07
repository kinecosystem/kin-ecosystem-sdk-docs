---
id: common-flows
sidebar_label: Common Flows
title: Common Flows
---
The following sections details how to implement some common workflows using the SDK.

<div class="toggler">

  <ul role="tablist" >
   <li id="android" class="button-android" aria-selected="false" role="tab" tabindex="0" aria-controls="androidtab" onclick="displayTab('platform', 'android')">
      Android
    </li>
    <li id="ios" class="button-ios" aria-selected="false" role="tab" tabindex="-1" aria-controls="iostab" onclick="displayTab('platform', 'ios')">
      iOS
    </li>
  </ul>
</div>

<block class="android ios" />

## Custom Spend Offer

A custom Spend offer allows your users to unlock unique spend opportunities that you define within your app, Custom offers are created by your app, as opposed to built-in offers displayed in the Kin Marketplace offer wall.  
Your app displays the offer, request user approval, and then performing the purchase using the `purchase` API.

### Purchase Payment

1. Create a JWT that represents a [Spend offer JWT](jwt#SpendPayload) signed by your application server. The fastest way for building JWT tokens is to use the [JWT Service](jwt-service).  
Once you have the JWT Service set up, perform a [Spend query](jwt-service#Spend),
the service will response with the generated signed JWT token.

2. Call `purchase` method, while passing the JWT you built and a callback function that will receive purchase confirmation.

> The following snippet is taken from the SDK Sample App, in which the JWT is created and signed by the Android client side for presentation purposes only. Do not use this method in production! In production, the JWT must be signed by the server, with a secure private key.

<block class="android" />
```java
try {
    Kin.purchase(offerJwt, new KinCallback<OrderConfirmation>() {
        @Override public void onResponse(OrderConfirmation orderConfirmation) {
            // OrderConfirmation will be called once Kin received the payment transaction from user.
            // OrderConfirmation can be kept on digital service side as a receipt proving user received his Kin.
            // Send confirmation JWT back to the server in order prove that the user completed
            // the blockchain transaction and purchase can be unlocked for this user.
            System.out.println("Succeed to create native spend.\n jwtConfirmation: " + orderConfirmation.getJwtConfirmation());                
        }

        @Override
        public void onFailure(KinEcosystemException exception) {
            System.out.println("Failed - " + error.getMessage());
        }
    });
} catch (ClientException e) {
    e.printStackTrace();
}
```

<block class="ios" />
```swift
Kin.shared.purchase(offerJWT: encodedNativeOffer) { jwtConfirmation, error in
  if let confirm = jwtConfirmation {
    // jwtConfirmation can be kept on digital service side as a receipt proving user received his Kin.
    // Send confirmation JWT back to the server in order prove that the user completed
    // the blockchain transaction and purchase can be unlocked for this user.
  } else if let e = error {
    // handle error
  }
}
```

<block class="android ios" />

3.	Complete the purchase after you receive confirmation from the Kin Server that the funds were transferred successfully.

### Adding to the Marketplace 
The Kin Marketplace offer wall displays built-in offers, which are served by Kin.  
Their purpose is to provide users with opportunities to earn initial Kin funding, which they can later spend on spend offers provided by hosting apps.

You can also choose to display a banner for your custom offer in the Kin Marketplace offer wall. This serves as additional "real estate" in which to let the user know about custom offers within your app. When the user clicks on your custom Spend offer in the Kin Marketplace, your app is notified, and then it continues to manage the offer activity in its own UX flow.

>**NOTE:** You will need to actively launch the Kin Marketplace offer wall so your user can see the offers you added to it.

<block class="android" />

1. Create a `NativeSpendOffer` object as in the example below.

```java
NativeSpendOffer nativeOffer =
        new NativeSpendOffer("The offerID") // An unique offer id
            .title("Offer Title") // Title to display with offer
            .description("Offer Description") // Description to display with offer
            .amount(100) // Purchase amount in Kin
            .image("Image URL"); // Image to display with offer
```

2.	Create a `NativeOfferObserver` object to be notified when the user clicks on your offer in the Kin Marketplace.

>**NOTE:** You can remove the Observer by calling `Kin.removeNativeOfferClickedObserver(...)`.

```java
private void addNativeOfferClickedObserver() {
    try {
        Kin.addNativeOfferClickedObserver(getNativeOfferClickedObserver());
    } catch (TaskFailedException e) {
        showToast("Could not add native offer callback");
    }
}

private Observer<NativeSpendOffer> getNativeOfferClickedObserver() {
    if (nativeSpendOfferClickedObserver == null) {
        nativeSpendOfferClickedObserver = new Observer<NativeSpendOffer>() {
            @Override
            public void onChanged(NativeSpendOffer value) {
                Intent nativeOfferIntent = NativeOfferActivity.createIntent(MainActivity.this, value.getTitle());
                startActivity(nativeOfferIntent);
            }
        };
    }
    return nativeSpendOfferClickedObserver;
}
```

3. Call `Kin.addNativeOffer(...)`.

>**NOTE:** Each new offer is added as the first offer in Spend Offers list the Marketplace displays.

```java
try {
    if (Kin.addNativeOffer(nativeSpendOffer)) {
        // Native offer added
    } else {
        // Native offer already in Kin Marketplace
    }
} catch (ClientException error) {
    ...
}
```

<block class="ios" />

1. Create a ```NativeSpendOffer``` struct as in the example below.

```swift
let offer = NativeOffer(id: "offer id", // OfferId must be a UUID
                        title: "offer title",
                        description: "offer description",
                        amount: 1000,
                        image: "an image URL string",
                        isModal: true)
```
> Note: setting a native offer's `isModal` property to true means that when a user taps on the native offer, the marketplace will first close (dismiss) before invoking the native offer's handler, if set. The default value is false.

2.	Set the  `nativeOfferHandler` closure on Kin.shared to receive a callback when the native offer has been tapped.  
The callback is of the form `public var nativeOfferHandler: ((NativeOffer) -> ())?`

```swift
// example from the sample app:
Kin.shared.nativeOfferHandler = { offer in
            DispatchQueue.main.async {
                let alert = UIAlertController(title: "Native Offer", message: "You tapped a native offer and the handler was invoked.", preferredStyle: .alert)
                alert.addAction(UIAlertAction(title: "Close", style: .cancel, handler: { [weak alert] action in
                    alert?.dismiss(animated: true, completion: nil)
                }))

                let presentor = self.presentedViewController ?? self
                presentor.present(alert, animated: true, completion: nil)
            }
        }
```

3. Add the native offer you created in the following way:

>Note: Each new offer is added as the first offer in Spend Offers list the Marketplace displays.

```swift
do {
    try Kin.shared.add(nativeOffer: offer)
} catch {
    print("failed to add native offer, error: \(error)")
}
```

<block class="android ios" />
### Removing from Marketplace

<block class="android" />
To remove a custom Spend offer from the Kin Marketplace, call `Kin.removeNativeOffer(...)`, passing the offer you want to remove.  

```java
try {
    if (Kin.removeNativeOffer(nativeSpendOffer)) {
        // Native offer removed
    } else {
        // Native offer isn't in Kin Marketplace
    }
} catch (ClientException e) {
    ...
}
```

<block class="ios" />

To remove a custom Spend offer from the Kin Marketplace, call `Kin.shared.remove(...)`, passing the offer you want to remove.  

```swift
do {
    try Kin.shared.remove(nativeOfferId: offer.id)
} catch {
    print("Failed to remove offer, error: \(error)")
}
```

<block class="android ios" />

## Custom Earn Offer

A custom Earn offer allows your users to earn Kin as a reward for performing tasks you want to incentives, such as setting a profile picture or rating your app. Custom offers are created by your app, as opposed to built-in offers displayed in the Kin Marketplace offer wall.  
Once the user has completed the task associated with the Earn offer, you request Kin payment for the user.

### Request A Payment

1. Create a JWT that represents a [Earn offer JWT](jwt#EarnPayload) signed by your application server. The fastest way for building JWT tokens is to use the [JWT Service](jwt-service).  
Once you have the JWT Service set up, perform a [Earn query](jwt-service#Earn),
the service will response with the generated signed JWT token.

2. Call `requestPayment` while passing the JWT you built and a callback function that will receive purchase confirmation.

>* The following snippet is taken from the SDK Sample App, in which the JWT is created and signed by the Android client side for presentation purposes only. Do not use this method in production! In production, the JWT must be signed by the server, with a secure private key.     

<block class="android" />
```java
try {
    Kin.requestPayment(offerJwt, new KinCallback<OrderConfirmation>() {
        @Override
            public void onResponse(OrderConfirmation orderConfirmation) {
                // Callback will be called once payment transaction to the user completed successfully.
                // jwtConfirmation can be kept on digital service side as a receipt proving user received his Kin.
                System.out.println("Succeed to create native earn.\n jwtConfirmation: " + orderConfirmation.getJwtConfirmation());
            }

            @Override
            public void onFailure(KinEcosystemException exception) {
                System.out.println("Failed - " + exception.getMessage());
            }
        });
}
catch (ClientException exception) {
    exception.printStackTrace();
}
```

<block class="ios" />

```swift
let handler: ExternalOfferCallback = { jwtConfirmation, error in  
    let alert = UIAlertController(title: nil, message: nil, preferredStyle: .alert)
    if let confirm = jwtConfirmation {
        // Callback will be called once payment transaction to the user completed successfully.
        // jwtConfirmation can be kept on digital service side as a receipt proving user received his Kin.
    } else if let e = error {
        //handle error
    }  
}

Kin.shared.requestPayment(offerJWT: encodedJWT, completion: handler)
```

<block class="android ios" />

## Custom Pay To User Offer

A custom pay to user offer allows your users to unlock unique spend opportunities that you define within your app offered by other users.
(Custom offers are created by your app, as opposed to built-in offers displayed in the Kin Marketplace offer wall.  
Your app displays the offer, request user approval, and then performing the purchase using the `payToUser` API.

### Pay to user

*To request payment for a custom Pay To User offer:*

1. Create a JWT that represents a [Pay to User offer JWT](jwt#PayToUserPayload) signed by your application server. The fastest way for building JWT tokens is to use the [JWT Service](jwt-service).  
Once you have the JWT Service set up, perform a [Pay To User query](jwt-service#PayToUser),
the service will response with the generated signed JWT token.


2.	Call `Kin.payToUser(...)`, while passing the JWT you built and a callback function that will receive purchase confirmation.

> The following snippet is taken from the SDK Sample App, in which the JWT is created and signed by the Android client side for presentation purposes only. Do not use this method in production! In production, the JWT must be signed by the server, with a secure private key. 

<block class="android" />
```java
try {
    Kin.payToUser(offerJwt, new KinCallback<OrderConfirmation>() {
        @Override public void onResponse(OrderConfirmation orderConfirmation) {
            // OrderConfirmation will be called once Kin received the payment transaction from user.
            // OrderConfirmation can be kept on digital service side as a receipt proving user received his Kin.
            // Send confirmation JWT back to the server in order prove that the user completed
            // the blockchain transaction and purchase can be unlocked for this user.
            System.out.println("Succeed to create native spend.\n jwtConfirmation: " + orderConfirmation.getJwtConfirmation());                
        }

        @Override
        public void onFailure(KinEcosystemException exception) {
            System.out.println("Failed - " + error.getMessage());
        }
    });
} catch (ClientException e) {
    e.printStackTrace();
}
```

<block class="ios" />
```swift
Kin.shared.payToUser(offerJWT: encodedNativeOffer) { jwtConfirmation, error in
  if let confirm = jwtConfirmation {
    // jwtConfirmation can be kept on digital service side as a receipt proving user received his Kin.
    // Send confirmation JWT back to the server in order prove that the user completed
    // the blockchain transaction and purchase can be unlocked for this user.
  } else if let e = error {
    // handle error
  }
}
```

<block class="android ios" />

3.	Complete the pay to user offer after you receive confirmation from the Kin Server that the funds were transferred successfully.

## Requesting an Order Confirmation

In the normal flow of a transaction, you will receive an order confirmation from the Kin Server through the offers APIs callback function. This indicates that the transaction was completed. But if you missed this notification for any reason, for example, because the user closed the app before it arrived, or the app closed due to some error, you can request confirmation for an order according to its ID.

<block class="android" />

Call `Kin.getOrderConfirmation(...)`, while passing the order’s ID and implementing the appropriate callback functions.

```java
try {
    Kin.getOrderConfirmation("your_offer_id", new KinCallback<OrderConfirmation>() {
            @Override
            public void onResponse(OrderConfirmation orderConfirmation) {
                if(orderConfirmation.getStatus() == Status.COMPLETED ){
                    String jwtConfirmation = orderConfirmation.getJwtConfirmation()
                }
            }

            @Override
            public void onFailure(KinEcosystemException exception) {
                ...
            }
    });
} catch (ClientException exception) {
    ...
}
```

<block class="ios" />

TBD

<script>
  function displayTab(type, value) {
    var container = document.getElementsByTagName('block')[0].parentNode;
    container.className = 'display-' + type + '-' + value + ' ' +
      container.className.replace(RegExp('display-' + type + '-[a-z]+ ?'), '');
  }
  function convertBlocks() {
    // Convert <div>...<span><block /></span>...</div>
    // Into <div>...<block />...</div>
    var blocks = document.querySelectorAll('block');
    for (var i = 0; i < blocks.length; ++i) {
      var block = blocks[i];
      var span = blocks[i].parentNode;
      var container = span.parentNode;
      container.insertBefore(block, span);
      container.removeChild(span);
    }
    // Convert <div>...<block />content<block />...</div>
    // Into <div>...<block>content</block><block />...</div>
    blocks = document.querySelectorAll('block');
    for (var i = 0; i < blocks.length; ++i) {
      var block = blocks[i];
      while (
        block.nextSibling &&
        block.nextSibling.tagName !== 'BLOCK'
      ) {
        block.appendChild(block.nextSibling);
      }
    }
  }
  function guessPlatformAndOS() {
    if (!document.querySelector('block')) {
      return;
    }
    // If we are coming to the page with a hash in it (i.e. from a search, for example), try to get
    // us as close as possible to the correct platform and dev os using the hashtag and block walk up.
    var foundHash = false;
    if (
      window.location.hash !== '' &&
      window.location.hash !== 'content'
    ) {
      // content is default
      var hashLinks = document.querySelectorAll(
        'a.hash-link'
      );
      for (
        var i = 0;
        i < hashLinks.length && !foundHash;
        ++i
      ) {
        if (hashLinks[i].hash === window.location.hash) {
          var parent = hashLinks[i].parentElement;
          while (parent) {
            if (parent.tagName === 'BLOCK') {
              // Could be more than one target os and dev platform, but just choose some sort of order
              // of priority here.
              // Target Platform
              if (parent.className.indexOf('ios') > -1) {
                displayTab('platform', 'ios');
                foundHash = true;
              } else if (
                parent.className.indexOf('android') > -1
              ) {
                displayTab('platform', 'android');
                foundHash = true;
              } else {
                break;
              }
            }
            parent = parent.parentElement;
          }
        }
      }
    }
    // Do the default if there is no matching hash
    if (!foundHash) {
      var isMac = navigator.platform === 'MacIntel';
      var isWindows = navigator.platform === 'Win32';
      displayTab('platform', isMac ? 'ios' : 'android');
    }
  }
  convertBlocks();
  guessPlatformAndOS();
</script>

<style>
  .toggler li {
    display: inline-block;
    position: relative;
    top: 1px;
    padding: 10px;
    margin: 0px 2px 0px 2px;
    border: 1px solid #05A5D1;
    border-bottom-color: transparent;
    border-radius: 3px 3px 0px 0px;
    color: #05A5D1;
    background-color: transparent;
    font-size: 0.99em;
    cursor: pointer;
  }
  .toggler li:first-child {
    margin-left: 0;
  }
  .toggler li:last-child {
    margin-right: 0;
  }
  .toggler ul {
    width: 100%;
    display: inline-block;
    list-style-type: none;
    margin: 0;
    border-bottom: 1px solid #05A5D1;
    cursor: default;
  }
  @media screen and (max-width: 960px) {
    .toggler li,
    .toggler li:first-child,
    .toggler li:last-child {
      display: block;
      border-bottom-color: #05A5D1;
      border-radius: 3px;
      margin: 2px 0px 2px 0px;
    }
    .toggler ul {
      border-bottom: 0;
    }
  }
  .toggler a {
    display: inline-block;
    padding: 10px 5px;
    margin: 2px;
    border: 1px solid #05A5D1;
    border-radius: 3px;
    text-decoration: none !important;
  }
  .display-platform-ios .toggler .button-ios,
  .display-platform-android .toggler .button-android {
    background-color: #05A5D1;
    color: white;
  }
  block { display: none; }
  .display-platform-ios .ios,
  .display-platform-android .android {
    display: block;
  }
</style>