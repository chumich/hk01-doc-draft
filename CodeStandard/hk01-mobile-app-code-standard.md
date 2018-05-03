# `Drafting ... Work In Progress`
## `hk01-mobile-app` Code Standard

The following are some suggested code standard for the `hk01-mobile-app` [repo](https://github.com/hk01-digital/hk01-mobile-app/).

## Table of Content

- [1. Prefer destructuring assignment](#1-prefer-destructuring-assignment)
- [2. Prefer the 'Or' version when using Ramda functions](#2-prefer-the-or-version-when-using-ramda-functions)
- [3. Prefered import style when using Ramda](#3-prefered-import-style-when-using-ramda)
- [4. Prefer fat arrow function over .bind(this)](#4-prefer-fat-arrow-function-over-bindthis)
- [5. Enclose plain text in JSX with tags](#5-enclose-plain-text-in-jsx-with--tags)
- [6. Use PropTypes from 'prop-types'](#6-use-proptypes-from-prop-types)

---
### 1. Prefer destructuring assignment

#### What

use [destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) to increase readability:
 
```
const { articleId, mainCategory } = this.props
```
instead of using `this.props.articleId` and `this.props.mainCategory`

* even if you are only using `articleId` and nothing else from `this.props`, it is still a good idea to use destructuring assignment

#### Why

* there is a tendency for future maintainer to follow **existing code** so hopefully with destructuring assignment is in place, there is less risk of seeing a lot of `this.props.xxx` and `this.props.yyy` everywhere

* from [this discussion](https://stackoverflow.com/questions/48454454/react-props-destructuring-and-memory-usage) it seems that there is no performance / memory overhead

---
### 2. Prefer the 'Or' version when using Ramda functions

#### What

* prefer `pathOr()` over `path()`

* prefer `propOr()` over `prop()`

#### Why

* the `'Or' version` requires a `default value` to be supplied, it forces the user to consider the possibility when the target is not found

* by explicitly specifying and handling the `default value` instead of hoping the returned value won't be `Undefined`, this would hopefully reduce potential crash points

---
### 3. Prefered import style when using Ramda

#### What

prefer the following import style:

```
import * as R from 'ramda'

//...

R.propOr(...)
```
instead of:

```
const propOr = require('ramda/src/propOr')

//...

propOr(...)
```
#### Why

purely aesthetic reason, it looks cleaner

---
### 4. Prefer fat arrow function over .bind(this)

#### What

if you have something looks like:

```
onItemPress = item => { ... }

// ... 

// In some render func:
// (assume item is passed into it)

<NotificationItem
  onPress={this.onItemPress.bind(this, item)}
/>
```
it is suggested to use fat arrow function instead of .bind(this):

```
<NotificationItem
  onPress={(item) => { this.onItemPress(item) }}
/>
```

#### Why

purely aesthetic reason, it is more readable

---
### 5. Enclose plain text in JSX with <Text> tags

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

---
### 6. Use PropTypes from 'prop-types'

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


---