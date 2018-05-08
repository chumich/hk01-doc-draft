| Version | Last Updated |
|---------|--------------|
| Draft.1 | 2018-05-08   |

# Data Tracking - Best Practice

The following is a *so-called 'best practice'* for adding data tracking in `hk01-mobile-app`.

The guide is just a suggested approach and certainly not definitely, suggestions are welcomed.

---

### NOTICE
* We will be shortly refactoring `toGaPiwikData` logic into `trackEvent`
* This guide will be updated after refactoring

---

## Table of Content

`Work in Progress`

`Work in Progress`

`Work in Progress`

## TL;DR;

1. Use `tracker` from `context` or `props`
2. Use `trackScreenView` for screen view event
3. Use `trackEvent` for other events
4. Do **not** use `trackScreenViewWithDimension` and `fire` - they will be removed
5. Use `EVT_CATEGORY` and `EVT_ACTION` 
6. Transform data using `toGaPiwikData` for event message

Example:

```
// TODO: change import route
import { EVT_CATEGORY, EVT_ACTION } from 'App/Constants/tracking'
import { toGaPiwikData } from 'App/Helpers/TrackingHelper'

// create your event message
const eventMessage = {
  'article_id': articleId
  'category': articleCategory
  'account_id': accountId
  'bucket_id': appBucketId
}

// send event tracking
this.context.tracker.trackEvent(
  EVT_CATEGORY.ARTICLE,
  EVT_ACTION.SHARE_CLICK,
  toGaPiwikData(eventMessage) 
}

// NOTE: toGaPiwikData() logic will later be moved into trackEvent()

```

## Getting Tracker

* The `tracker` is initialized in `App/Containers/index.js` and passed into `context` by `AppProvider`
* In some Container/Component it might be wrapped into `props` by `withAppConfig`
* Generally it is more preferrable to directly use the `tracker` from `context`

You should be able to use the `tracker` from either `context` or `props`:

```
this.context.tracker.trackEvent(...)

// or

this.props.tracker.trackScreenView(...)

```

## Track Screen View

* Use only the `trackScreenView` function

```
trackScreenView (screenName: string)
```

* Avoid using `trackScreenViewWithCustomDimension` as it will be later removed

```
trackScreenViewWithCustomDimensions (
  screenName: string,
  customDimensions: any
)
```

* See `App/Constants/screenName.js` for a list of `SCREEN_NAME` available
* Add a new screen name under `SCREEN_NAME` with reference to [Data Tracking Spec](#more-information) if needed

## Track Event

* Use only the `trackEvent` function

```
trackEvent (
  category: string, 
  action: string, 
  message: Object
)
```

* Avoid using `fire` function as it will be later removed

### Event Category and Action

* The event **categories** and **actions** are defined in `App/Constants/tracking.js`
* The actual `string` value can be cross referenced from the [Data Tracking Spec](#more-information)
* Add new value(s) under `EVT_CATEGORY` and/or `EVT_ACTION` if needed

**NEVER** directly pass `string` value to category/action

```
// NEVER DO THIS
this.context.tracker.trackEvent('ArticleView', 'CommentIcon')

// DO THIS INSTEAD
this.context.tracker.trackEvent(
  EVT_CATEGORY.ARTICLE_VIEW,
  EVT_ACTION.TAP_ARTICLEVIEW_COMMENTICON
)
```

#### Naming

* Use `CAPITAL` letter with underscore `_` separator when adding new values under `EVT_CATEGORY` and/or `EVT_ACTION`
* Naming should be base on the actual `string` value with separation at the `CamelCase`

```
// GOOD
DEVICE_ID: 'DeviceId'

// BAD
DEVICEID: 'DeviceId'
```

### Event Message

* For events without a event message, just omit the `message` parameter
* If you have a event message, use `toGaPiwikData` helper function in `TrackingHelper` to transform it before passing to `trackEvent`

```
// App/Helpers/TrackingHelper.js

export const joinObjectValues = (data: Object): ?string => {
  return R.compose(R.join('/'), R.values)(data)
}

export const toGaPiwikData = (data: Object): ?Object => {
  return {
    [TRACKER.GA]: joinObjectValues(data),
    [TRACKER.PIWIK]: data
  }
}
```
For example the following event message:

```
{
  'channelName': '熱門',
  'articleId': '163761',
  'pressedItemPosition': '1',
  'appBucketId': '00001'
}
```
After data transformation:

```
{
  "ga": "熱門/163761/1/00001",
  "piwik": "{\"channelName\":\"熱門\",\"articleId\":\"163761\",\"pressedItemPosition\":\"1\",\"appBucketId\":\"00001\"}"
}
```

## More Information

This is the [Data Tracking Spec](https://docs.google.com/spreadsheets/d/13hlpwAst7UO81HabafIRdNqZoDp9Fa6-Zr8xdvF2JcA/edit#gid=690822871)

Following are some useful files to look, when trying to understand the whole data tracking thing

|What|File|
|---|---|
|`Tracker` Type|`App/types/tracker.js`|
|`Tracker` Service (Class)|`App/Services/Tracker.js`|
|`TrackingOperator` Helper|`App/Helpers/TrackingHelper.js`|
|Implementation of tracker for `GA`|`App/Services/Tracker/GA.js`|
|Implementation of tracker for `Piwik`|`App/Services/Tracker/Piwik.js`|
|Defined tracking related constants|`App/Constants/tracking.js`|