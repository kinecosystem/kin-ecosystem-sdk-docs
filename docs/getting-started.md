---
id: getting-started
title: Getting Started
---

## Obtaining Authentication Credentials

### Whitelist

Receive your api key and app-id.

> **Note:** whitelist registration should be used only for a quick first-time integration or sanity test, see [Whitelist limitations](jwt#WhitelistNote).

### JWT

* Generate your JWT key pair.
* Provide Ecosystem with your JWT public keys.
* Receive your JWT issuer jss/app-id, that will uniquely identify your application.
  
## Integrate Client SDK

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
</br>

### Add Client SDK To Your App

<block class="android" />

Add the following lines to your project module's ```build.gradle``` file.
```gradle
 repositories {
     ...
     maven {
         url 'https://jitpack.io'
     }
 }
```
Add the following lines to the app module's ```build.gradle``` file.

```gradle
 dependencies {
     ...
     implementation 'com.github.kinfoundation:kin-ecosystem-android-sdk:<latest_version>'
 }
```

latest version can be found in [github releases](https://github.com/kinecosystem/kin-devplatform-android/releases).

<block class="ios" />

The fastest way to get started with the sdk is with cocoapods (>= 1.4.0).
```
pod 'KinDevPlatform', '<latest release>'
```
latest version can be found in [github releases](https://github.com/kinecosystem/kin-devplatform-ios/releases)

> Notice for apps using swift 3.2: the pod installation will change your project's swift version target to 4.0  
> This is because the sdk uses swift 4.0, and cocoapods force the pod's swift version on the project. For now, you can manually change your project's swift version in the build setting. A better solution will be available soon.

<block class="ios android" />
## Initializing The SDK

### Generating Registration JWT

The first step for Initializing the SDK is to build a [registration JWT token](jwt#RegisterPayload) at application server side, The fastest way for building JWT tokens is to use the [JWT Service](jwt-service).  
Once you have the JWT Service set up, perform a [Register query](jwt-service#Register),
the service will response with the generated signed JWT token, your app should be able to request this JWT generation on demand when SDK initialization is needed at the client side.

### Initialize Client SDK

<block class="android" />

Call ```Kin.start(…)```, passing the android context, the desired environment (playground/production) and your chosen authentication credentials (either whitelist or JWT credentials).

#### Whitelist:

```java
whitelistData = new WhitelistData(<userID>, <appID>, <apiKey>);
try {
   Kin.start(getApplicationContext(), whitelistData,
             Environment.getPlayground());
} 
catch (ClientException | BlockchainException e) {
   // Handle exception…
}
```

userID - your application unique identifier for the user  
appID - your application unique identifier as provided by Kin.  
apiKey - your secret apiKey as provided by Kin.

#### JWT:

Request a registration JWT from your server, once the client received this token, you can now start the sdk using this token.
```java
try {
    String registrationJWT = getRegistrationJwtFromServer();
    Kin.start(getApplicationContext(), registrationJWT, Environment.getPlayground());
}
catch (ClientException | BlockchainException e) {
   // Handle exception…
}
```

<block class="ios" />


Call ```Kin.shared.start(...)```, passing the desired environment (playground/production) and your chosen authentication credentials (either whitelist or JWT credentials).

#### Whitelist:

```swift
Kin.shared.start(userId: "myUserId", apiKey: "myAppKey", appId: "myAppId", environment: .playground)
```

userID - your application unique identifier for the user  
appID - your application unique identifier as provided by Kin.  
apiKey - your secret apiKey as provided by Kin.

#### jwt:

Request a registration JWT from your server, once the client received this token, you can now start the sdk using this token.

```swift
Kin.shared.start(userId: "myUserId", jwt: registrationJWT, environment: .playground)
```

<block class="ios android" />

This will create the stack needed for running the SDK, All account creation and activation is handled for you by the sdk.  
Because blockchain onboarding might take a few seconds, It is strongly recommended to call this function as soon as you can provide a user id.

### Launching The Marketplace

The final stage for activating a user is to launch the marketplace UI.  
In the first time the SDK is used, the user must accept the Terms of Service before doing any earn/spend opportunity.

<block class="android" />

For launching the Kin Marketplace offer wall, use `launchMarketplace` with an `Activity` object.

```java
try {
    Kin.launchMarketplace(activity);
        System.out.println("Public address : " + Kin.getPublicAddress());
} catch (ClientException e) {
    // handle exception...
}
```
<block class="ios">

To launch the Kin Marketplace offer wall, from a viewController, simply call:

```swift
Kin.shared.launchMarketplace(from: self)
```

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