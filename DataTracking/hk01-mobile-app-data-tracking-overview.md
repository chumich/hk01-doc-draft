| Verion | Last Updated |
|--------|--------------|
| Draft  | 2018-05-07   |

## hk01-mobile-app Data Tracking

### Useful Documents

[The Data Tracking Spec](https://docs.google.com/spreadsheets/d/13hlpwAst7UO81HabafIRdNqZoDp9Fa6-Zr8xdvF2JcA/edit#gid=690822871) maintained by the *Data Team* contains **category**, **action** and **message** (payload) for required events in the App.

### Overview

Currently we have data tracking events sent to [Google Analytics (GA)](https://analytics.google.com/) and [Piwik (Matomo)](https://matomo.org/), there is a stub for [Appsflyer](https://www.appsflyer.com/) but that might be reoved later.

#### Underlying Trackers

|Tracker|File|Using|
|---|---|---|
|GA |`App/Services/Tracker/GA.js`|[react-native-piwik (self-maintained)](https://github.com/hk01-digital/react-native-piwik)|
|Piwik|`App/Services/Tracker/Piwik.js`|[react-native-google-analytics-bridge (5.1.0)](https://github.com/idehub/react-native-google-analytics-bridge/releases)|
|Appsflyer|`App/Services/Tracker/Appsflyer.js`|`stub only`|

#### Tracker Definition

The following definition for tracker is often referred to as the `TrackerType`

```
// App/types/tracker.js

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
|trackScreenView|`Yes`|`Yes`|
|trackScreenViewWithCustomDimensions|`Yes`|**`No`**|
|trackEvent|`Yes`|`Yes`|
|fire|`*`|`*`|

##### Note

_*_`fire` is an alternative route for `trackEvent` and `trackScreenView` for some **category** and **action** combination

* see `App/Helpers/TrackingHelper.js` for details
* see `EVT_TRACK_NAME` in `App/Constants/tracking.js`

### TrackScreenView and TrackScreenViewWithCustomDimensions

These two functions are fairly straight forward, see `App/Constants/screenName.js` for a list of `SCREEN_NAME` available to be used.

#### Function signature

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

### TrackEvent

#### Function Signature

```
trackEvent (
  category: string, 
  action: string, 
  message: Object
): void
```

##### Note
It is worth noting that, apart from `category` and `action`, the underlying `trackEvent` signature in the actual `Piwik` or `GA` SDK is expecting a `label` (`string`) and `value` (`number`) pair.

In both cases, only the `string` part is used. 


#### Event Category and Event Action

* The event categories and actions are defined in `App/Constants/tracking.js`
* These are used as parameters for `trackEvent` function
* The actual `string` definition can be cross referenced from the [Data Tracking Spec](https://docs.google.com/spreadsheets/d/13hlpwAst7UO81HabafIRdNqZoDp9Fa6-Zr8xdvF2JcA/edit#gid=690822871)

```
// App/Constants/tracking.js

// event category
export const EVT_CATEGORY = {
  ABOUT_US: 'AboutUsView',
  ADVERTISEMENT: 'AD',
  //...
}

// event action
export const EVT_ACTION = {
  //...
  TAP_CONTACTUS: 'TapContactUs',
  //...
  DEVICEID: 'DeviceId'
  //...
}
```

#### Event Message

Note that not all events have a message, for example the following event:

```
trackEvent(EVT_CATEGORY.TAB_BAR, EVT_ACTION.TAP_HOME)
```

For the events that have a message, it might look something like:

```
{
  "channelName": "熱門",
  "articleId": "163761",
  "pressedItemPosition": "1",
  "appBucketId": "00001"
}
```

##### Difference in message format

There is a difference in the message format for `Piwik` and `GA`.

Take the above message as an example, `Piwik` will receive a string representation of the message object:

```
"{\"channelName\":\"熱門\",\"articleId\":\"163761\",\"pressedItemPosition\":\"1\",\"appBucketId\":\"00001\"}"
```

whereas `GA` will get a string with all the values in the message object joined with `/` separator:

```
"熱門/163761/1/00001"
```

##### Actual implementation

**`Piwik`** implementation takes the value from the `piwik` key and invokes `JSON.stringify` on it.

**`GA`** implementation, on the other hand, takes value from the `ga` key and invokes `.toString()` on the value.

When calling `trackEvent` with a message object, make sure the message should look like the following:

```
{
  "ga": "熱門/163761/1/00001",
  "piwik": "{\"channelName\":\"熱門\",\"articleId\":\"163761\",\"pressedItemPosition\":\"1\",\"appBucketId\":\"00001\"}"
}
```

There is a helper function in `TrackingHelper` which can be used for the transformation


```
// App/Helpers/TrackingHelper.js

export const toGaPiwikData = (data: Object): ?Object => {
  return {
    [TRACKER.GA]: joinObjectValues(data),
    [TRACKER.PIWIK]: data
  }
}
```

##### Proposed improvement - Option 1

If there is no objection from the *Data Team* or other teams that might be using GA data, we could keep the message send to both `GA` and `Piwik` the same.

i.e. call `JSON.stringify` for the `GA` part too, keep the message the same and there will be no need for the `toGaPiwikData`.

**Potential issue**

* data limit for GA server side?
* need to check if some other teams (editorial?) are using GA `/` separated data

##### Proposed improvement - Option 2

Use `toGaPiwikData` in `trackEvent` to transform the message payload if key `ga` and `piwik` are not found.

This avoid code like:

```
switch (listType) {
  //...
  case COMMON_LIST_TYPE.HOT_ARTICLE:
    itemId = this.getItemId(currentItem)
    channelName = I18n.t('popular')
    categorySuffix = EVT_MODIFIERS.v2Suffix
    gaLabel =
      channelName + '/' + itemId + '/' + itemPosition + '/' + appBucketId
    piwikValue = { channelName, itemId, itemPosition, appBucketId }
    break
  //...
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



---
# WORK IN PROGRESS BELOW
---

### Problems

See the following example, there are a couple of problems

1. legacy `gaLabel`
2. data only send to `GA`
3. hardcoded value for `eventCategory`

```
// App/Components/Article/ActionBar/index.js

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
* `gaLabel` might be legacy or some instances of usage are incorrect
* the content in eventValue is ignored in current implementation anyway
* suggest to refactor after tests are in place 

#### event value useless / only for GA
* the value `'CommentIcon'` is identical to `EVT_ACTION.TAP_ARTICLEVIEW_COMMENTICON`
* it is only send to `GA` but not `Piwik`
* it is seemingly useless anyway
* suggest to remove after tests are in place

#### hardcoded value for `eventCategory`
* `eventCategory` **must** use value from `EVT_CATEGORY` enum instead of hardcoding the `string` value
* suggest to refactor after tests are in place

### Context or Props

* tracker can come from `this.context` or `this.props`
* might not need to change

```
// tracker initialized in 
```

#### Problem: different entry points
* use of `fire()`, `trackEvent()`, `trackScreenView()` and `trackScreenViewWithCustomDimensions()`
* presence of `trackingSaga` and `trackingRedux`


#### Tracker Service


#### Special case: Tracking Saga

* Needed for tracking at API call back - mostly wallet side
* This is a better approach as we don't need to add a bunch of flags in `componenetDidMount` / `componentDidUpdate` to check whether the API call has succeeded or failed.



### Suggested 'Best Practice'

1. Less points of entry, use only `trackEvent` and `trackScreenView` and avoid `fire` and `trackScreenViewWithCustomDimensions`
2. Consumer of the Tracker service passes the message object `eventValue` (if any), transformation for `GA` and `Piwik` is done inside `trackEvent` and `trackScreenView`
3. The transformation will check if the payload already contains `ga` and `piwik` key, if so then no transformation will be done, this also allows a bit flexibility
4. Keep existing impementation and refactor after tests are implemented
5. Implement tests and refactor


### Current approach in `TrackingHelper`

##### Pros

1. Might be easier for testing (spy on a single function call and check parameters)
2. Data tracking firing and payload combining is centralised in one file

#### Cons

1. Need to maintain the arbitary `EVT_TRACK_NAME` which is an extra overhead
2. High possibility of conflict when multiple teams are trying to add data tracking



### Need for Tests

Ultimately we want to have **one or more tests** for every data tracking event fired, screen view event included. 

Initial idea is to use `sinon` to spy on `fire()` of `TrackingHelper` to make sure the function is invoked with expected `EVT_CATEGORY`, `EVT_ACTION` and payload (if any).

For the user interaction currently we are looking into detox to see if that is a feasible way for implementing the tests.