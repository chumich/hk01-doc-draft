---
id: webview-platform-specific
title: WebView - Platform Specific
---

# WebView - Platform Specific

## TL; DR;

* iOS implementation of `WebView` component in **react-native** uses `UIWebView`, however `WKWebView` is better
* use `/App/Components/WebView/WebViewPlatformSpecific/` which uses `WKWebView` from [`react-native-wkwebview`](https://github.com/CRAlpha/react-native-wkwebview)
* see [Caveats](caveats) section for important points to note

## The Problem

Apple recommended using `WKWebView` over `UIWebView` [as of iOS 8](https://9to5mac.com/2014/06/03/ios-8-webkit-changes-finally-allow-all-apps-to-have-the-same-performance-as-safari/) for [improved JS performance](https://developer.telerik.com/featured/why-ios-8s-wkwebview-is-a-big-deal-for-hybrid-development/).

However the iOS side implementation of `WebView` in **react-native** is still using `UIWebview`. [The discussion](https://github.com/facebook/react-native/issues/321) for switching to `WKWebView` started in Mar 2015, and is still on-going as of May 2018.

> Honestly, we, at Facebook, do not use this component so much, so it is unlikely that I or one of my colleagues will have time to dive into this, so... PRs is welcome!
> 
> — [shergin commented on 18 Aug 2017](https://github.com/facebook/react-native/issues/321#issuecomment-323262908)

So, it is unlikely that the `UIWebView`-based `WebView` is going be updated any soon.

### Related Issues

There are a few issues that are due to the (relatively) outdated `UIWebView`, for example:

* [CORE-47](https://hk01-digital.atlassian.net/browse/CORE-47)
* [CORE-90](https://hk01-digital.atlassian.net/browse/CORE-90)

## The Solution

The [`react-native-wkwebview`](https://github.com/CRAlpha/react-native-wkwebview) is a well-received and open-source library, it offers a drop-in replacement for `WKWebView` implementation and [is recommended](https://github.com/facebook/react-native/issues/321#issuecomment-222585962).

[CORE-151](https://hk01-digital.atlassian.net/browse/CORE-151) introduces a platform-specific `WebView` componenent `WebViewPlatformSpecific` which could be used to replace the current `WebView`.

> **NOTE**: The `WebViewPlatformSpecific` only affects **iOS**

To enjoy the benefits from `WKWebView`, start by changing this:
 
```JavaScript
import { WebView } from 'react-native'

```

To this:

```JavaScript
import WebView from 'App/Components/WebView/WebViewPlatformSpecific/'
```

See [Caveats](#caveats) section for points to note.

## Current Status

As of [CORE-151](https://hk01-digital.atlassian.net/browse/CORE-151), implementation of `App/Components/WebView/index.js` has been changed to use `WebViewPlatformSpecific `.

This in term affects **all** callers of `App/Containers/WebView`, which includes the navigation route `'WebView'`.

However there are other users which are directly using `WebView` componenet from **react-native**, see the following table:

| File | Status | Remark |
|------|--------|--------|
|`App/Components/Article/Block/EmbedHtml/index.js`|Changed|Tested with article `36480`|
|`App/Components/Article/Block/MotionGraphic/index.js`|Changed|Tested with article `36480`|
|`App/Modules/Ticketing/Components/ZonesScreen/ZonesMap.js`|TBC|Potential problem with `scalesPageToFit`|
|`App/Modules/Ticketing/Containers/EventDescription/index.js`|TBC|Potential problem with `scalesPageToFit`|
|`App/Modules/Ticketing/Containers/EventScreen/index.js`|TBC|
|`App/Modules/Ticketing/Containers/MyTicketsScreen/index.js`|TBC|Potential problem with `scalesPageToFit`|
|`App/Modules/Ticketing/Containers/PaymentScreen/index.js`|TBC|
|`App/Modules/Ticketing/Containers/SeatsScreen/index.js`|TBC|Potential problem with `scalesPageToFit`, need to rename `injectJavaScript`|

## Caveats

### Unsupported Props

There are a few unsupported props in `WKWebView` that should be noted:

|Unsupported Props|Remarks|
|-----------------|-------|
| mediaPlaybackRequiresUserAction | not used in `hk01-mobile-app` |
| scalesPageToFit | **potential problem** |
| domStorageEnabled | android only, no concern |
| javaScriptEnabled | android only, no concern |
| allowsInlineMediaPlayback | not used in `hk01-mobile-app` |
| decelerationRate | iOS only, only `normal` is used so should not be a problem |

#### `scalesPageToFit`

Unable to use this props **might** create problem when relying it to display a `html` document correctly based on the device screen width.

One [potential solution](https://stackoverflow.com/questions/49121466/disable-zoom-in-webview-or-react-native-wkwebview-in-react-native) is to embed the following in the `html` document to be displayed:

```HTML
<meta content='width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0' name='viewport' />
```

#### `injectJavascript`

`injectJavascript` is renamed to `evaluateJavascript `

```JavaScript
// Change this:
this.webview.injectJavaScript(script)

// To this:
this.webview.evaluateJavaScript(script)
```

### WebView Events

In theory all events implemented in the in-house [`react-native-webview-events`](https://github.com/hk01-digital/react-native-webview-events) should be bridged back to App correctly.

See implementation in `/App/Containers/WebView/ConnectedWebView.js`.