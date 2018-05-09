| Version | Last Updated |
|---------|--------------|
| Draft.2 | 2018-05-09   |

# Data Tracking - Best Practice

The following is a *so-called 'best practice'* for adding new data tracking in `hk01-mobile-app`.

The guide is just a suggested approach and certainly not definitive, suggestions are welcomed.

> if you have suggestions, please open PR with edited version of this document

---

### NOTICE
* We will be shortly refactoring `toGaPiwikData` logic into `trackEvent`
* This guide will be updated afterwards

---

## Table of Content

1. [TL; DR;](#tl-dr)
2. [Getting Tracker](#getting-tracker)
3. [Track Screen View](#track-screen-view)
4. [Track Event](#track-event)
5. [Writing Test](#writing-test)
6. [Tracking Saga](#tracking-saga)
7. [More Information](#more-information)

## TL; DR;

1. Use `tracker` from `context` or `props`
2. Use `trackScreenView` for screen view event
3. Use `trackEvent` for other events
4. Do **not** use `trackScreenViewWithDimension` and `fire` - they will be removed
5. Use `EVT_CATEGORY` and `EVT_ACTION` 
6. Transform data using `toGaPiwikData` for event message

Example:
> NOTE: change the import route

```
import { EVT_CATEGORY, EVT_ACTION } from 'App/Constants/tracking'
import { toGaPiwikData } from 'App/Helpers/TrackingHelper'

// logic to prepare the payload content ...

// create your event message
const eventMessage = toGaPiwikData({
  'article_id': articleId
  'category': articleCategory
  'account_id': accountId
  'bucket_id': appBucketId
})

// send event tracking
this.context.tracker.trackEvent(
  EVT_CATEGORY.ARTICLE,
  EVT_ACTION.SHARE_CLICK,
  eventMessage 
}

```
> NOTE: `toGaPiwikData()` logic will later be moved into `trackEvent()`

## Getting Tracker

* The `tracker` is initialized in `App/Containers/index.js` and passed into `context` by `AppProvider`
* In some Container/Component it might be wrapped into `props` by `withAppConfig`
* Generally it is more preferrable to directly use the `tracker` from `context`

> You should be able to use the `tracker` from either `context` or `props`:

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
TAP_CONTACT_US: 'TapContactUs'

// BAD
DEVICEID: 'DeviceId'
TAP_CONTACTUS: 'TapContactUs'
```

### Event Message

* For events without a event message, omit the `message` parameter
* If a event message is needed, use `toGaPiwikData` helper function in `TrackingHelper` to transform it before passing to `trackEvent`

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
For example the following event payload:

```
{
  "channelName": "熱門",
  "articleId": "163761",
  "pressedItemPosition": "1",
  "appBucketId": "00001"
}
```
After data transformation:

```
{
  "ga": "熱門/163761/1/00001",
  "piwik": "{\"channelName\":\"熱門\",\"articleId\":\"163761\",\"pressedItemPosition\":\"1\",\"appBucketId\":\"00001\"}"
}
```

## Writing Test

`Work in Progress` `Work in Progress` `Work in Progress`

## Tracking Saga

In case of needing to send data tracking at every API call, one solution is to create `TrackingSaga`.

Following are some examples for reference on the `TrackingSaga` approach.

|Tracking Saga|Tracking Redux|
|-------------|--------------|
|`App/Modules/Wallet/Sagas/TrackingSagas.js`|`App/Modules/Wallet/Redux/TrackingRedux.js`|
|`App/Modules/Merchant/Sagas/TrackingSagas.js`|`App/Modules/Merchant/Redux/TrackingRedux.js`|
|`App/Modules/Ticketing/Sagas/TrackingSagas.js`|`App/Modules/Ticketing/Redux/TrackingRedux.js`|


## More Information

This is the [Data Tracking Spec](https://docs.google.com/spreadsheets/d/13hlpwAst7UO81HabafIRdNqZoDp9Fa6-Zr8xdvF2JcA/edit#gid=690822871)

More details on data tracking can be found in the [overview document]()
