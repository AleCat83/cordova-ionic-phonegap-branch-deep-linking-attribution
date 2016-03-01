#Cordova-Ionic-PhoneGap-Deferred-Deep-Linking-SDK/README.md

> Branch Metrics Cordova SDK

## What is this?

This Cordova plugin allows you to call Branch.IO API Endpoints, this shares almost the same code base as the Branch Web SDK.

## Register your Branch.IO app

You can sign up for your own Branch key at [https://dashboard.branch.io](https://dashboard.branch.io).

## Installation

There are multiple ways to add the plugin in to your app.

Thru Cordova

```sh
cordova plugin install branch-cordova-sdk --variable BRANCH_LIVE_KEY=your-branch-key --variable URI_SCHEME=your-app-uri-scheme --variable ENCODED_ID=your-encoded-id
```

Thru Phonegap

```sh
phonegap plugin add branch-cordova-sdk --variable BRANCH_LIVE_KEY=your-branch-key --variable URI_SCHEME=your-app-uri-scheme --variable ENCODED_ID=your-encoded-id
```

**Note:**
* `BRANCH_LIVE_KEY` - Your Branch.io live API key.
* `URI_SCHEME` - It could be your app name or the URI set in your Branch dashboard.
* `ENCODED_ID` - You only need to know  which you can obtain from the Branch dashboard once you enable App Linking support for your application. Please see [this](https://github.com/BranchMetrics/Android-Deferred-Deep-Linking-SDK/blob/master/README.md#leverage-android-app-links-for-deep-linking) section of the Android readme.

Thru NPM
```sh
npm install branch-cordova-sdk --variable BRANCH_LIVE_KEY=your-branch-key --variable URI_SCHEME=your-app-uri-scheme --variable ENCODED_ID=your-encoded-id
```

## Additional App Permissions

To be able to use some of the deep linking capabilities of the app, some manifest files are needed to be configured.
Please note that you don't have to do anything anymore for setting up the `Register a URI Scheme and add your Branch key`
as this was already done for you automatically on installing the plugin.

#### iOS: Enable Universal Links

In iOS 9.2, Apple dropped support for URI scheme redirects. You must enable Universal Links if you want Branch-generated links to work in your iOS app. To do this:

1. enable `Associated Domains` capability on the Apple Developer portal when you create your app's bundle identifier.
2. In your [Dashboard Link Settings](https://dashboard.branch.io/#/settings/link), tick the `Enable Universal Links` checkbox and provide the Bundle Identifier and Apple Team ID in the appropriate boxes.
3. Finally, add `associated-domains` to your entitlements file. Since cordova doesn't have a way to a create entitlements and associate it to your generated project, we
will generate the said file with the help of [Cordova Universal Links Plugin](https://github.com/nordnet/cordova-universal-links-plugin), a third plarty plugin.

**Note:** The purpose of the said plugin is to generate an entitlements file and associate it to your generated project. No other implementations from the plugin are need as
this guide will cover what only needs to be implemented.

To start, go to your project root and install the plugin:
```sh
cordova plugin add cordova-universal-links-plugin
```

After the installation, add the following entry to your application's `config.xml`:
```xml
<universal-links>
    <ios-team-id value=your_ios_team_id />
    <host name="bnc.lt">
    </host>
</universal-links>
```

You can get your iOS Team ID from the Apple Developer Portal.

Once done, you have successfully enabled universal links for iOS.

## Demo App

This repo includes a testbed app, that demonstrates all the features of the plugin. Please refer to the `README` inside the `testbed` folder.

---------------

## Plugin Methods

**Some methods are promisified**, therefore you can easily get its success and error callback by chaining `.then()` method.

*Example*
```js
Branch.getFirstReferringParams().then(function (res) {
  // Success Callback
  console.log(res);
}, function (err) {
  // Error Callback
  console.error(err);
});
```

1. Branch Session
  + [setDebug](#setDebug)
  + [initSession](#initSession)
  + [getLatestReferringParams](#getLatestReferringParams)
  + [getFirstReferringParams](#getFirstReferringParams)
  + [setIdentity](#setIdentity)
  + [logout](#logout)
  + [userCompletedAction](#userCompletedAction)
2. Branch Universal Object
  + [createBranchUniversalObject](#createBranchUniversalObject)
  + [registerView](#registerView)
  + [generateShortUrl](#generateShortUrl)
  + [showShareSheet](#showShareSheet)
  + [listOnSpotlight](#listOnSpotlight) **iOS only**
3. Referral System Rewarding
  + [loadRewards](#loadRewards)
  + [redeemRewards](#redeemRewards)
  + [creditHistory](#creditHistory)


### <a id="setDebug"></a>setDebug(isEnable)

Setting the SDK debug flag will generate a new device ID each time the app is installed instead of possibly using the same device id.
This is useful when testing.

**Parameters**

**options**: `boolean` - Boolean flag if debug mode should be enabled or not.

##### Usage
```js
Branch.setDebug(true);
```

### <a id="initSession"></a>initSession()

Initializes the branch instance.
**Note:** `setDebug()` should be called first before calling this method.

##### Usage
The `initSession()` method automatically also sets an internal deep link hander whose data
can be accesed by implementing the **required** `DeepLinkHandler()` method.

To implement, first call the method:
```js
Branch.initSession();
```
then add the method `DeepLinkHandler()` which will act as our callback for the response:
```js
function DeepLinkHandler(data)
{
    alert('Data from initSession: ' + data.data);

}
```

### <a id="getFirstReferringParams"></a>getFirstReferringParams()

Retrieves the install session parameters.

##### Usage
```js
Branch.getFirstReferringParams().then(function (res) {
  // Success Callback
  console.log(res);
}).catch(function (err) {
  // Error Callback
  console.error(err);
});
```

### <a id="getLatestReferringParams"></a>getLatestReferringParams()

Retrieves the session (install or open) parameters.

##### Usage
```js
Branch.getLatestReferringParams().then(function (res) {
  // Success Callback
  console.log(res);
}).catch(function (err) {
  // Error Callback
  console.error(err);
});
```

### <a id="setIdentity"></a>setIdentity(object)

Sets the identity of a user and returns the data. To use this function,
pass a unique string that identifies the user - this could be an email address, UUID, Facebook ID, etc.

**Parameters**

**identity**: `string` - A string uniquely identifying the user, ofetn a user ID or email address.

##### Usage
```js
Branch.setIdentity("new_identity").then(function (res) {
  // Success Callback
  console.log(res);
}).catch(function (err) {
  // Error Callback
  console.error(err);
});
```

### <a id="logout"></a>logout()

Logs out the current session, replaces session IDs and identity IDs.

##### Usage
```js
Branch.logout();
```

### <a id="userCompletedAction"></a>userCompletedAction(action[, metaData])

Registers custom events.

**Parameters**

**action**: `string` - A string for your custom action (e.g. "completed_purchase", "wrote_message", etc.)

**metaData**: `object` _[Optional]_ - Custom values to be passed with the action

##### Usage
```js
Branch.userCompletedAction('complete_purchase');

Branch.userCompletedAction('registered', { user: 'Test' });
```
------

## Branch Universal Object

As more methods evolved in iOS, we've found that it was increasingly hard to manage them all.
We abstracted as many as we could into the concept of a Branch Universal Object.
This is the object that is associated with the thing you want to share (content or user).
You can set all the metadata associated with the object and then call action methods on it to get a link or index in Spotlight.

### <a id="createBranchUniversalObject"></a>createBranchUniversalObject(object)

Initializes the universal Branch object.

**Parameters**

**options**: `object` - Options in creating object.

##### object

|         Key         | TYPE   |             DESCRIPTION             |
| ------------------- | ------ | ----------------------------------- |
| canonicalIdentifier | String | The object identifier               |
| title               | String | The object title                    |
| contentDescription  | String | Object Description                  |
| contentImageUrl     | String | The Image URL                       |
| contentIndexingMode | String | Indexing Mode 'private' or 'public' |
| contentMetadata     | Object | Custom key/value                    |

##### Usage
```js
var branchUniversalObj = null;

Branch.createBranchUniversalObject({
  canonicalIdentifier: 'identifier',
  title: 'Just another title',
  contentDescription: 'Just another description',
  contentImageUrl: '/img.jpg',
  contentIndexingMode: 'public'
}).then(function (newBranchUniversalObj) {
  // Success Callback
  branchUniversalObj = newBranchUniversalObj;
  console.log(newBranchUniversalObj);
}, function (err) {
  // Error Callback
  console.error(err);
});
```

### <a id="registerView"></a>registerView()

If you want to track how many times a user views a particular piece of content, you can call this method in `viewDidLoad` or `viewDidAppear`
to tell Branch that this content was viewed.

##### Usage
```js
branchUniversalObj.registerView();
```

### <a id="generateShortUrl"></a>generateShortUrl(options, controlParameters)

Once you've created your `Branch Universal Object`, which is the reference to the content you're interested in, you can then get a link back to it with the mechanism described below.

**Parameters**

**options**: `object` - Options needed to generate the URL.

|    KEY   |   TYPE   |          MEANING
| -------- | -------- |------------------------
| feature  | `string` | The feature of the link
| alias    | `string` | The alias of the link
| channel  | `string` | The channel of the link
| stage    | `string` | The stage of the link
| duration |  `int`   | duration of the link.

**controlParameters**: `object` - Link properties needed to generate the URL.

|        KEY         |   TYPE   |       MEANING
| ------------------ | -------- | --------------------
| $fallback_url      | `string` | The fallback URL
| $desktop_url       | `string` | The URL for desktop
| $android_url       | `string` | The URL for Android
| $ios_url           | `string` | The URL for iPhone
| $ipad_url          | `string` | The URL for iPad
| $fire_url          | `string` | The URL for Kindle Fire
| $blackberry_url    | `string` | The URL for Blackberry
| $windows_phone_url | `string` | The URL for Windows phone

##### Usage
```js
branchUniversalObj.generateShortUrl({
  "feature" : "sample-feature",
  "alias" : "sample-alias",
  "channel" : "sample-channel",
  "stage" : "sample-stage"
}, {
  "$desktop_url" : "http://desktop-url.com",
}).then(function (res) {
    // Success Callback
    console.log(res.generatedUrl);
}, function (err) {
    // Error Callback
    console.error(err);
});
```

### <a id="showShareSheet"></a>showShareSheet(options, controlParameters)

UIActivityView is the standard way of allowing users to share content from your app.
Once you've created your `Branch Universal Object`, which is the reference to the content you're interested in, you can then automatically share it _without having to create a link_ using the mechanism below.

**Sample Android Share Sheet**

![Android Share Sheet](https://dev.branch.io/img/ingredients/sdk_links/android_share_sheet.png)

**Sample UIActivityView Share Sheet**

![UIActivityView Share Sheet](https://dev.branch.io/img/ingredients/sdk_links/ios_share_sheet.jpg)

The Branch iOS SDK includes a wrapper on the UIActivityViewController, that will generate a Branch short URL and automatically tag it with the channel the user selects (Facebook, Twitter, etc.).

**Parameters**

**options**: `object` - Options needed to generate the URL.

|    KEY   |   TYPE   |          MEANING
| -------- | -------- |------------------------
| feature  | `string` | The feature of the link
| alias    | `string` | The alias of the link
| channel  | `string` | The channel of the link
| stage    | `string` | The stage of the link
| duration |  `int`   | duration of the link.

**controlParameters**: `object` - Link properties needed to generate the URL.

|        KEY         |   TYPE   |       MEANING
| ------------------ | -------- | --------------------
| $fallback_url      | `string` | The fallback URL
| $desktop_url       | `string` | The URL for desktop
| $android_url       | `string` | The URL for Android
| $ios_url           | `string` | The URL for iPhone
| $ipad_url          | `string` | The URL for iPad
| $fire_url          | `string` | The URL for Kindle Fire
| $blackberry_url    | `string` | The URL for Blackberry
| $windows_phone_url | `string` | The URL for Windows phone

##### Usage
```js
branchUniversalObj.showShareSheet({
  "feature" : "sample-feature",
  "alias" : "sample-alias",
  "channel" : "sample-channel",
  "stage" : "sample-stage",
  "duration" : 1,
}, {
  "$desktop_url" : "http://desktop-url.com",
});
```

##### Share Sheet Callbacks (Android ONLY)

To implement the callback, you must add listeners to the following events:

###### onShareSheetLaunched

The event fires when the share sheet is presented.

```js
branchUniversalObj.onShareSheetLaunched(function () {
  console.log('Share sheet launched');
});
```

###### onShareSheetDismissed

The event fires when the share sheet is dismissed.

```js
branchUniversalObj.onShareSheetDismissed(function () {
  console.log('Share sheet dimissed');
});
```

###### onLinkShareResponse

The event returns a dictionary of the response data.

```js
branchUniversalObj.onLinkShareResponse(function (res) {
  console.log('Share link response: ' + JSON.stringify(res));
});
```

###### onChannelSelected

The event fires when a channel is selected.

```js
branchUniversalObj.onChannelSelected(function (res) {
  console.log('Channel selected: ' + JSON.stringify(res));
});
```

**Note:** Callbacks in iOS are ignored. There is no need to implement them as the events are handled by `UIActivityViewController`.

**Note:** Avoid passing `alias` in iOS. Adding an `alias` key in the `options` parameter will return a Non-Universal link which will not work in iOS 9.2.

### <a id="listOnSpotlight"></a>listOnSpotlight()

**Note: iOS only.** Used for Spotlight listing

##### Usage
```js
branchUniversalObj.listOnSpotlight().then(function (res) {
  // Success Callback
  console.log(res);
}).catch(function (err) {
  // Error Callback
  console.error(err);
});
```

-------

## Referral System Rewarding

### <a id="loadRewards"></a>loadRewards()

Reward balances change randomly on the backend when certain actions are taken (defined by your rules), so you'll need to make an asynchronous call to retrieve the balance. Here is the syntax:

##### Usage
```js
Branch.loadRewards().then(function (rewards) {
  // Success Callback
  console.log(rewards);
}).catch(function (err) {
  // Error Callback
  console.error(err);
});
```

### <a id="redeemRewards"></a>redeemRewards(value)

Redeems a reward with the given amount/value.

**Parameters**

|    KEY   |   TYPE   |          MEANING
| -------- | -------- |------------------------
| value  | `int` | Amount to be redeemed.
| bucket    | `int` | Bucket where the amount will be redeemed. _optional_

##### Usage
```js
Branch.redeemRewards(100, "default").then(function (res) {
  // Success Callback
  console.log(res);
}).catch(function (err) {
  // Error Callback
  console.error(err);
});
```

### <a id="creditHistory"></a>creditHistory()

This call will retrieve the entire history of credits and redemptions from the individual user. To use this call, implement like so:

##### Usage
```js
Branch.creditHistory().then(function (history) {
  // Success Callback
  console.log(history);
}, function (err) {
  // Error Callback
  console.error(err);
});
```

The response will return an array that has been parsed from the following JSON:

```js
[
    {
        "transaction": {
                           "date": "2014-10-14T01:54:40.425Z",
                           "id": "50388077461373184",
                           "bucket": "default",
                           "type": 0,
                           "amount": 5
                       },
        "event" : {
            "name": "event name",
            "metadata": { your event metadata if present }
        },
        "referrer": "12345678",
        "referree": null
    },
    {
        "transaction": {
                           "date": "2014-10-14T01:55:09.474Z",
                           "id": "50388199301710081",
                           "bucket": "default",
                           "type": 2,
                           "amount": -3
                       },
        "event" : {
            "name": "event name",
            "metadata": { your event metadata if present }
        },
        "referrer": null,
        "referree": "12345678"
    }
]
```

**referrer**

The id of the referring user for this credit transaction. Returns null if no referrer is involved. Note this id is the user id in developer's own system that's previously passed to Branch's identify user API call.

**referree**

The id of the user who was referred for this credit transaction. Returns null if no referree is involved. Note this id is the user id in developer's own system that's previously passed to Branch's identify user API call.

**type**

This is the type of credit transaction
1. 0 - A reward that was added automatically by the user completing an action or referral
2. 1 - A reward that was added manually
3. 2 - A redemption of credits that occurred through our API or SDKs
4. 3 - This is a very unique case where we will subtract credits automatically when we detect fraud

## Bugs / Help / Support

Feel free to report any bugs you might encounter in the repo's issues. Any support inquiries outside of bugs
please send to [support@branch.io](mailto:support@branch.io).
