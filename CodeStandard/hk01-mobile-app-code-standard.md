# `hk01-mobile-app` Code Standard

The following are some suggested code standard for the `hk01-mobile-app` [repo](https://github.com/hk01-digital/hk01-mobile-app/).

## Table of Content

- [0. General](#general)
- [1. Prefer destructuring assignment](#prefer-destructuring-assignment)
- [2. Prefer the 'Or' version when using Ramda functions](#prefer-the-or-version-when-using-ramda-functions)
- [3. Prefered import style when using Ramda](#preferred-import-style-when-using-ramda)
- [4. Prefer fat arrow function over .bind(this)](#prefer-fat-arrow-function-over-bindthis)


## General

### Boy Scout Rule

Please follow the [Boy Scout Rule](http://deviq.com/boy-scout-rule/)

![Boy Scout Rule](https://lh3.googleusercontent.com/-eM1_28qE1cM/U1bUFmBU1NI/AAAAAAAAHEk/ZqLcxFEhMuA/w530-h398-n/slide-32-638.jpg)

* Please avoid leaving unnecessary `TODO`, commented out code blocks, etc. in the codebase.

> The act of leaving a mess in the code should be as socially unacceptable as littering. 
> 
> — [Robert C. “Uncle Bob” Martin](http://programmer.97things.oreilly.com/wiki/index.php/The_Boy_Scout_Rule)

### Linting

[Javascript Standard Style](https://standardjs.com/rules.html) is used for linting.

## String Literals

#### What

Define and use constants for string literals under `App/Constants/`.

**Avoid** doing something like:

```Javascript
if (displayMode === 'bigImage') {
  // ...
} else if (displayMode === 'threeImage') {
  // ...
```

Instead, define the string literals **if they have not yet been defined**:

> e.g. in `App/Constants/DisplayMode.js`

```Javascript
export const DISPLAY_MODE = {
  BIG_IMAGE: 'bigImage',
  THREE_IMAGE: 'threeImage',
  RIGHT_IMAGE: 'rightImage',
  // ...
}
```

And use them as such:

```Javascript
import { DISPLAY_MODE } from 'App/Constants/DisplayMode'

// ...

if (displayMode === DISPLAY_MODE.BIG_IMAGE) {
  // ...
} else if (displayMode === DISPLAY_MODE.THREE_IMAGE) {
  // ...
```
* Use `SNAKE_CASE` for constant names
* Use `camelCase` for newly created string literals if possible

#### Why

It is better to centralise definition of string literals and use them to avoid multiple definition such as `'Text'` `'TEXT'` `'text'` which would create confusion and lower the maintainability of the codebase.

#### Exception

When writing tests there is no need to define string literals as specified.

## Type Definitions

#### What

Define and use types under `App/types/`.

**Avoid** defining type in individual files:

> // e.g. in `App/Components/ImageTitleRow/index.js`

```Javascript
type DisplayMode = 'rightImage' | 'bigImage' | 'threeImage'

type Props = {
  displayMode?: DisplayMode,
  // ...
```

Instead, define the type in a separate file:

> e.g. in `App/types/DisplayMode.js`

```Javascript
import { DISPLAY_MODE } from 'App/Constants/DisplayMode'

export type DisplayMode =
  | DISPLAY_MODE.RIGHT_IMAGE
  | DISPLAY_MODE.BIG_IMAGE
  | DISPLAY_MODE.THREE_IMAGE
```

And use as such:

```Javascript
import type { DisplayMode } from 'App/types/DisplayMode'

type Props = {
  displayMode?: DisplayMode,
  // ...
```


## Prefer destructuring assignment

#### What

Use [destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) for increased readability:
 
```Javascript
const { articleId, mainCategory } = this.props
```
instead of using `this.props.articleId` and `this.props.mainCategory`

* even if you are only using `articleId` and nothing else from `this.props`, it is still a good idea to use destructuring assignment

#### Why

* there is a tendency for future maintainer to follow **existing code** so hopefully with destructuring assignment is in place, there is less risk of seeing a lot of `this.props.xxx` and `this.props.yyy` everywhere

* from [this discussion](https://stackoverflow.com/questions/48454454/react-props-destructuring-and-memory-usage) it seems that there is no performance / memory overhead

## Prefer the 'Or' version when using Ramda functions

#### What

* prefer `pathOr()` over `path()`

* prefer `propOr()` over `prop()`

#### Why

* the **'Or' version** requires a `default value` to be supplied, it forces the user to consider the possibility when the target is not found

* by explicitly specifying and handling the `default value` instead of hoping the returned value won't be `undefined`, this would hopefully reduce potential crash points


## Preferred import style when using Ramda

#### What

> Use the following import style:

```Javascript
import * as R from 'ramda'

//...

R.propOr(...)
```
> But **NOT** this:

```Javascript
const propOr = require('ramda/src/propOr')

//...

propOr(...)
```
#### Why

Purely aesthetic reason, it looks cleaner


## Prefer fat arrow function over .bind(this)

#### What

Assume we have a callback `onItemPress` that we want to pass to `NotificationItem`

```Javascript
onItemPress = item => { ... }
```

> Use fat arrow function:

```
<NotificationItem
  onPress={(item) => { this.onItemPress(item) }}
  // other props ...
/>
```

> Instead of .bind(this):

```
<NotificationItem
  onPress={this.onItemPress.bind(this, item)}
  // other props ...
/>
```

#### Why

Purely aesthetic reason, it is more readable

## React Native Caveats

The following are not exactly code standards, but some caveats that should be avoided when dealing with ReactNatived

### Enclose plain text in JSX with <Text> tags

#### What

instead of writing:

```
<Button>Press Me</Button>
```
enclose the plain text in <Text></Text> tags like so:

```
<Button>
  <Text>Press Me</Text>
</Button>
```

#### Why

as in [this discussion](https://stackoverflow.com/questions/47627231/cannot-add-a-child-that-doesnt-have-a-yoganode-to-a-parent-without-a-measure-fu), plain text that are not properly enclosed in JSX will likely lead to the following crash:

> Cannot add a child that doesn't have a YogaNode to a parent without a measure function! (Trying to add a 'ReactRawTextShadowNode' to a 'LayoutShadowNode')


also see [this thread](https://github.com/facebook/react-native/issues/13243) for other potential crash points that would result in the crash message:

> Cannot Add a child that doesn't have a YogaNode to a parent with out a measure function

### Use PropTypes from 'prop-types'

#### What

when using `PropTypes`, instead of:

```
import { PropTypes } from 'react'
```

use:

```
import PropTypes from 'prop-types'
```


#### Why

this is a [requirement from React](https://reactjs.org/docs/typechecking-with-proptypes.html)

> React.PropTypes has moved into a different package since React v15.5. Please use the prop-types library instead.
