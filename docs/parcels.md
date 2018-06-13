# Parcels

A single-spa parcel is a bundle of functionality (like an application or component) meant to be mounted manually 
by an application. Parcels use similar methodology as applications but are mounted by a manual function call rather than the activity function.
A parcel can be as large as an application or as small as an component and written in 
any language as long as it exports the correct lifecycle events. In a single-spa world, your SPA contains 
many registered applications and potentially many parcels. Typically we recommend you mount a parcel within 
the context of an application because the parcel will be unmounted with the application.

## Parcel configuration
A parcel is just an object with 3 or 4 functions on it, it can be loaded syncronously or asyncronously.
Each function is considered a lifecycle method, `bootstrap`, `mount`, `unmout`, `update` and must return a promise.
When creating a parcel it's strongly recommended that you use the [lifecycle helper methods](/docs/single-spa-ecosystem.md#help-for-frameworks).
An example of a parcel written in React would look like this:
```js
// myParcel.js
import React from 'react'
import ReactDom from 'react-dom'
import singleSpaReact from 'single-spa-react'
import MyParcelComponent from './my-parcel-component.component.js'
export const MyParcel = singleSpaReact({
  React,
  ReactDom,
  rootComponent: MyParcelComponent
})

// in this case singleSpaReact is taking our inputs and generating an object with the required lifecycles.
```
Then to use the parcel you just created all you need to do is use the `Parcel` component provided in single-spa-react
```js
// mycomponent.js
import { Parcel } from 'single-spa-react'
import MyParcel from './myparcel.js'

export myComponent extends React.Component {
    render () {
      return (
        <Parcel
          config={MyParcel}
          { /* and any extra props you want here */ }
        />

      )
    }
}
```

## Parcel Lifecycles
Start with [applications](/docs/applications.md#registered-application-lifecycle) to learn more about the functionality of single-spa's lifecycle methods.

### Bootstrap
This lifecycle function will be called once, right before the parcel is
mounted for the first time.

```js
function bootstrap(props) {
  return Promise
    .resolve()
    .then(() => {
        // This is where you do one-time initialization
        console.log('bootstrapped!')
        });
}
```

### Mount
If the parcel is not mounted this lifecycle function is called when ever `mountParcel` is called. When 
called, this function should create DOM elements, DOM event listeners, etc. to render content to the user.

```js
function mount(props) {
  return Promise
    .resolve()
    .then(() => {
        // This is where you tell a framework (e.g., React) to render some ui to the dom
        console.log('mounted!')
        });
}
```

### Unmount
This lifecycle function will be called whenever the parcel is mounted and one of the following cases is true:
- `unmount()` is called
- The parent parcel or application is unmounted
When called, this function should clean up all DOM elements, DOM event listeners, leaked memory, globals, 
observable subscriptions, etc. that were created at any point when the parcel was mounted.

```js
function unmount(props) {
  return Promise
    .resolve()
    .then(() => {
        // This is where you tell a framework (e.g., React) to unrender some ui from the dom
        console.log('unmounted!');
        });
}
```

### Update (optional)
This lifecycle function will be called whenever the parcel is mounted, and it needs to be updated.

## Example use cases

### Modals
`App1` handles everything related to contacts (highly cohesive) but somewhere in `App2` we need to create a contact.
We could do any number of things to share the functionality between application 1 and 2: 
- If both are written in the same framework we could export/import components.
- We could reimplement creating a contact (loss of cohesion)
- We could use single-spa parcels.

Exporting a parcel from `App1` that wraps the createContact modal component gives us the ability to share components and behavior across disparate frameworks, without losing application cohesion.
App1 can export a modal as a single-spa parcel and App2 can import the parcel and use it easily. One major advantage is that in the below example
the parcel/modal from App1 that is being used by App2 will also be unmounted, without unmounting/mounting of App1.

```js
// App1
export const AddContactParcel = {
  bootstrap: bootstrapFn,
  mount: mountFn,
  unmount: unmountFn,
}

// App2
// get the parcel configuration in this case I'm using systemJS and react
...
componentDidMount() {
  SystemJS.import('App1').then(App1 => {
    const domElement = document.body
    App2MountProps.mountParcel(App1.AddContactParcel, {domElement})
  })
}
...

```

### Cross Library Components

There are a lot of different approaches to share components across libraries. One we use extensively 
for leaf nodes (buttons, etc) is web components. However once you move beyond a leaf node it's much more difficult 
to share complex components across different frameworks. Single Spa parcels takes the same concepts of multiple 
applications written in multiple frameworks and applies that to components.

### Imperative Applications

Sometimes you need a single spa application to be mounted in a very specific situation. That situation 
could be in an area where another application is active 99% of the time by itself. You could use the [application activity Function](/docs/single-spa-config.md#activity-function)
and `localStorage` or `history` to manage that or you could mount your application as a parcel.
