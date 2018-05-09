| Version | Last Updated |
|---------|--------------|
| Draft.3 | 2018-05-09   |

# Data Tracking - Overview

This document aims at giving an overview on data tracking in `hk01-mobile-app`.

If you are looking for **suggested best practice**, take a look at [this document](hk01-mobile-app-data-tracking-best-practice.md) instead.

## Tabel of Content

1. [Useful Information](#useful-information)
2. [Overview](#overview)
3. [`trackScreenView` and `trackScreenViewWithCustomDimensions`](#trackscreenview-and-trackscreenviewwithcustomdimensions)
4. [`trackEvent`](#trackevent)
5. [Proposed Improvement](#proposed-improvement)
6. [Legacy Code Study](#legacy-code-study)

## Useful Information

[The Data Tracking Spec](https://docs.google.com/spreadsheets/d/13hlpwAst7UO81HabafIRdNqZoDp9Fa6-Zr8xdvF2JcA/edit#gid=690822871) maintained by the *Data Team* defines **category**, **action** and **message** for required events in the App.

When trying to understand data tracking, following are some useful files to dig into:

|What|File|
|----|----|
|`Tracker` Type|`App/types/tracker.js`|
|`Tracker` Service (Class)|`App/Services/Tracker.js`|
|`TrackingOperator` Helper|`App/Helpers/TrackingHelper.js`|
|Implementation of tracker for `GA`|`App/Services/Tracker/GA.js`|
|Implementation of tracker for `Piwik`|`App/Services/Tracker/Piwik.js`|
|Defined tracking related constants|`App/Constants/tracking.js`|

## Overview

Currently we have data tracking events sent to [Google Analytics (GA)](https://analytics.google.com/) and [Piwik (Matomo)](https://matomo.org/), there is a stub for *AppsFlyer* but that will be removed later.

### Tracker Implementation

The base implementation for trackers are using external libraries:

|Tracker|External Lib.|Version|
|-------|-------------|-------|
|GA|[react-native-piwik](https://github.com/hk01-digital/react-native-piwik)|self-maintained|
|Piwik|[react-native-google-analytics-bridge](https://github.com/idehub/react-native-google-analytics-bridge/releases)|5.1.0|


### Tracker Definition

The following definition for tracker is often referred to as the `TrackerType`
> App/types/tracker.js

```
type Tracker = {
  init: Function,
  setShowDebugLog: Function,
  setAddDebugMessageHandler: Function,
  trackScreenView: Function,
  trackScreenViewWithCustomDimensions: Function,
  trackEvent: Function,
  fire: Function
}
```
The functions that are useful for data tracking are as follow:

|Function|GA |Piwik|
|--------|---|-----|
|`trackScreenView`|`Yes`|`Yes`|
|`trackScreenViewWithCustomDimensions`|`Yes, to be removed`|**`No`**|
|`trackEvent`|`Yes`|`Yes`|
|`fire`|`Yes*`|`Yes*`|


> *`fire` is an alternative route for `trackEvent` and `trackScreenView` for some combination of **category** and **action**

* See `App/Helpers/TrackingHelper.js` for details
* See `EVT_TRACK_NAME` in `App/Constants/tracking.js`
* This approach is target to be removed in later version

## `trackScreenView` and `trackScreenViewWithCustomDimensions`

These two functions are fairly straight forward, see `App/Constants/screenName.js` for a list of `SCREEN_NAME` available.

### Function Signature

```
trackScreenView (screenName: string): string
```
```
trackScreenViewWithCustomDimensions (
  screenName: string,
  customDimensions: any
): string
```

* It should be noted that the `trackScreenViewWithCustomDimensions` is **only available** for `GA` implementation
* The `Piwik` implementation for `trackScreenView` has return type `void`
* The returned `string` value for `GA` is the debug message that can be used for debug console display

## `trackEvent`

### Function Signature

```
trackEvent (
  category: string, 
  action: string, 
  message: Object
): void
```

> It is worth noting that, apart from `category` and `action`, the underlying `trackEvent` call in the actual Piwik or GA **SDK** is expecting a label and value pair which is `string` and `number` respectively.

> In both cases, only the `string` part is used. 


### Event Category and Event Action

The event categories `EVT_CATEGORY` and actions `EVT_ACTION` are defined in `App/Constants/tracking.js`

>// App/Constants/tracking.js

```
// event category
export const EVT_CATEGORY = {
  // ...
}

// event action
export const EVT_ACTION = {
  // ...
}
```

### Event Message

Note that not all events have a message, for example the following event:

```
trackEvent(EVT_CATEGORY.TAB_BAR, EVT_ACTION.TAP_HOME)
```

For the events that have a message, the payload might look something like:

```
{
  "channelName": "熱門",
  "articleId": "163761",
  "pressedItemPosition": "1",
  "appBucketId": "00001"
}
```

#### Difference in message format

There is a difference in the message format for `Piwik` and `GA`.

Take the above payload as an example, `Piwik` will receive a string representation of the payload JSON object:

```
"{\"channelName\":\"熱門\",\"articleId\":\"163761\",\"pressedItemPosition\":\"1\",\"appBucketId\":\"00001\"}"
```

whereas `GA` will get a `string` with all the values in the payload JSON object joined with `/` separator:

```
"熱門/163761/1/00001"
```

#### Tracker-side handling

**`Piwik`** implementation takes the value from the `piwik` key and invokes `JSON.stringify()` for transformation.

**`GA`** implementation, on the other hand, takes value from the `ga` key and invokes `.toString()`.

## Proposed Improvement

This section describes a suggested improvement for the `trackEvent` function in `App/Services/Tracker.js`

### 1. Event Message Data Transformation Logic

Move the logic of `toGaPiwikData` in `trackEvent` to transform the message payload **only if** the key `ga` and `piwik` are not found.

1. This can avoid a lot of duplication code to customise event message (`eventValue` in the below example)
2. This still allow certain degree of customisation for different payload for `GA` and `Piwik` if needed


```
switch (listType) {
  // ...
  case COMMON_LIST_TYPE.HOT_ARTICLE:
    itemId = this.getItemId(currentItem)
    channelName = I18n.t('popular')
    categorySuffix = EVT_MODIFIERS.v2Suffix
    gaLabel =
      channelName + '/' + itemId + '/' + itemPosition + '/' + appBucketId
    piwikValue = { channelName, itemId, itemPosition, appBucketId }
    break
  // ...
}

const eventValue = {
  `[TRACKER.GA]: gaLabel,
  `[TRACKER.PIWIK]: piwikValue
}

this.context.tracker.trackEvent(
  this.eventCategory + categorySuffix,
  EVT_ACTION.LOAD_MORE,
  eventValue
)
```

### 2. Add Tests

Ideally we want to have **one or more tests** for every data tracking event fired, screen view event included.

When the tests are in place we can confidently refactor/remove the existing/legacy code for data tracking without need to worry of missing tracking events.

Initial idea is to use `sinon` to spy on tracking functions of `Tracker` Service to make sure the function is invoked with expected `EVT_CATEGORY`, `EVT_ACTION` and message (if any).

For the user interaction currently we are looking into `detox` to see if that is a feasible way for implementing the tests.

## Legacy Code Study

See the [best practice document](hk01-mobile-app-data-tracking-best-practice.md) for how to add new data tracking, do **NOT** blindly following existing code as they might be legacy.

### Problems

See the following example, there are a couple of problems

1. Legacy `gaLabel`
2. Message payload only send to `GA`
3. Hardcoded value for `eventCategory`

> App/Components/Article/ActionBar/index.js

```
trackTapCommentIconEvent = () => {
  let eventValue
  let eventCategory = 'ArticleView'
  let articleIdWithTitle = ''
  if (this.props.article) {
    articleIdWithTitle =
      this.props.article.articleId + '' + this.props.article.title
  }
  
  eventValue = {
    gaLabel: articleIdWithTitle,
    [TRACKER.GA]: 'CommentIcon'
  }
  
  this.context.tracker.trackEvent(
    eventCategory,
    EVT_ACTION.TAP_ARTICLEVIEW_COMMENTICON,
    eventValue
  )
}
```

#### `gaLabel`? 
* `gaLabel` might be legacy, or some instances of usage are incorrect
* The content in `gaLabel` is ignored in current implementation anyway
* Suggest to remove after tests are in place 

#### Useless event message / only send to GA
* The value `'CommentIcon'` is identical to `EVT_ACTION.TAP_ARTICLEVIEW_COMMENTICON`
* It is only send to `GA` but not `Piwik`
* It is seemingly useless
* Suggest to remove after tests are in place

#### Hardcoded value for `eventCategory`
* `eventCategory` **must** use value from `EVT_CATEGORY` enum instead of hardcoding the `string` value
* suggest to refactor after tests are in place

