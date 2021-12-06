# Finch-native-template

## Authors

- **Atticus Emery** - [Like A Finch](https://github.com/like-a-finch)



## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Getting Started

#### 1. Clone and Install

```bash
# Clone the repo
git clone https://github.com/like-a-finch/finch-native.git

# Navigate to clonned folder and Install dependencies
cd finch-native && yarn install

gem install cocoapods


# Install Pods
cd ios && pod install
```

#### 2. Open App in your iOS simulator

Run this command to start the development server and to start your app on iOS simulator:
```
yarn run:ios
```

Or, if you prefer Android:
```
yarn run:android
```

# BI React/Redux Table of Contents

Teach The BI implementation of Redux/Redux

- [BI React/Redux Table of Contents](#bi-reactredux-table-of-contents)
	- [createSlice()](#createslice)
		- [Creating Slices of State](#creating-slices-of-state)
		- [Simplifying Slices with `createSlice`](#simplifying-slices-with-createslice)
		- [Exporting and Using Slices](#exporting-and-using-slices)
		- [Parameters](#parameters)
		- [`reducers`](#reducers)
			- [Customizing Generated Action Creators](#customizing-generated-action-creators)
			- [`initialState`](#initialstate)
			- [`name`](#name)
			- [`extraReducers`](#extrareducers)
			- [The "builder callback" API for `extraReducers`](#the-%22builder-callback%22-api-for-extrareducers)
			- [Return Value](#return-value)
		- [Examples](#examples)
	- [createSelector(...inputSelectors | [inputSelectors], resultFunc)](#createselectorinputselectors--inputselectors-resultfunc)
		- [`createSelector function` Examples](#createselector-function-examples)
- [React Hooks Overview](#react-hooks-overview)
	- [`useState`](#usestate)
			- [Functional updates](#functional-updates)
		- [Equivalent Class Example](#equivalent-class-example)
		- [Declaring a State Variable](#declaring-a-state-variable)
		- [Reading State](#reading-state)
		- [Updating State](#updating-state)
		- [Tip: What Do Square Brackets Mean?](#tip-what-do-square-brackets-mean)
		- [Tip: Using Multiple State Variables](#tip-using-multiple-state-variables)
	- [`useEffect`](#useeffect)
		- [Cleaning up an effect](#cleaning-up-an-effect)
		- [Timing of effects](#timing-of-effects)
		- [Conditionally firing an effect](#conditionally-firing-an-effect)
	- [`useCallback`](#usecallback)
	- [`useMemo`](#usememo)
	- [`useRef`](#useref)
	- [`useSelector()`](#useselector)
		- [Equality Comparisons and Updates](#equality-comparisons-and-updates)
		- [`useSelector` Examples](#useselector-examples)
			- [Using memoizing selectors](#using-memoizing-selectors)
	- [`useDispatch()`](#usedispatch)
			- [Examples](#examples-1)
	- [`Hooks Rules`](#hooks-rules)
		- [Only Call Hooks at the Top Level](#only-call-hooks-at-the-top-level)
		- [Only Call Hooks from React Functions](#only-call-hooks-from-react-functions)
		- [ESLint Plugin](#eslint-plugin)
		- [Explanation](#explanation)

## createSlice()

A function that accepts an initial state, an object full of reducer functions, and a "slice name",
and automatically generates action creators and action types that correspond to the reducers and state.

### Creating Slices of State

Redux state is typically organized into "slices", defined by the reducers that are passed to `combineReducers`:

```js
import { combineReducers } from 'redux'
import usersReducer from './usersReducer'
import postsReducer from './postsReducer'

const rootReducer = combineReducers({
  users: usersReducer,
  posts: postsReducer
})
```

In this example, both `users` and `posts` would be considered "slices". Both of the reducers:

- "Own" a piece of state, including what the initial value is
- Define how that state is updated
- Define which specific actions result in state updates

The common approach is to define a slice's reducer function in its own file, and the action creators in a second file. Because both functions need to refer to the same action types, those are usually defined in a third file and imported in both places:

```js
// postsConstants.js
const CREATE_POST = 'CREATE_POST'
const UPDATE_POST = 'UPDATE_POST'
const DELETE_POST = 'DELETE_POST'

// postsActions.js
import { CREATE_POST, UPDATE_POST, DELETE_POST } from './postConstants'

export function addPost(id, title) {
  return {
    type: CREATE_POST,
    payload: { id, title }
  }
}

// postsReducer.js
import { CREATE_POST, UPDATE_POST, DELETE_POST } from './postConstants'

const initialState = []

export default function postsReducer(state = initialState, action) {
  switch (action.type) {
    case CREATE_POST: {
      // omit implementation
    }
    default:
      return state
  }
}
```

The only truly necessary part here is the reducer itself. Consider the other parts:

- We could have written the action types as inline strings in both places.
- The action creators are good, but they're not _required_ to use Redux - a component could skip supplying a `mapDispatch` argument to `connect`, and just call `this.props.dispatch({type : "CREATE_POST", payload : {id : 123, title : "Hello World"}})` itself.
- The only reason we're even writing multiple files is because it's common to separate code by what it does

The ["ducks" file structure](https://github.com/erikras/ducks-modular-redux) proposes putting all of your Redux-related logic for a given slice into a single file, like this:

```js
// postsDuck.js
const CREATE_POST = 'CREATE_POST'
const UPDATE_POST = 'UPDATE_POST'
const DELETE_POST = 'DELETE_POST'

export function addPost(id, title) {
  return {
    type: CREATE_POST,
    payload: { id, title }
  }
}

const initialState = []

export default function postsReducer(state = initialState, action) {
  switch (action.type) {
    case CREATE_POST: {
      // Omit actual code
      break
    }
    default:
      return state
  }
}
```

That simplifies things because we don't need to have multiple files, and we can remove the redundant imports of the action type constants. But, we still have to write the action types and the action creators by hand.

### Simplifying Slices with `createSlice`

To simplify this process, Redux Toolkit includes a `createSlice` function that will auto-generate the action types and action creators for you, based on the names of the reducer functions you provide.

Here's how that posts example would look with `createSlice`:

```js
const postsSlice = createSlice({
  name: 'posts',
  initialState: [],
  reducers: {
    createPost(state, action) {},
    updatePost(state, action) {},
    deletePost(state, action) {}
  }
})

console.log(postsSlice)
/*
{
    name: 'posts',
    actions : {
        createPost,
        updatePost,
        deletePost,
    },
    reducer
}
*/

const { createPost } = postsSlice.actions

console.log(createPost({ id: 123, title: 'Hello World' }))
// {type : "posts/createPost", payload : {id : 123, title : "Hello World"}}
```

`createSlice` looked at all of the functions that were defined in the `reducers` field, and for every "case reducer" function provided, generates an action creator that uses the name of the reducer as the action type itself. So, the `createPost` reducer became an action type of `"posts/createPost"`, and the `createPost()` action creator will return an action with that type.

```js
const postsSlice = createSlice({
  name: 'posts',
  initialState: [],
  reducers: {
    createPost(state, action) {},
    updatePost(state, action) {},
    deletePost(state, action) {}
  }
})

const { createPost } = postsSlice.actions

console.log(createPost({ id: 123, title: 'Hello World' }))
// {type : "posts/createPost", payload : {id : 123, title : "Hello World"}}
```

### Exporting and Using Slices

Most of the time, you'll want to define a slice, and export its action creators and reducers. The recommended way to do this is using ES6 destructuring and export syntax:

```js
const postsSlice = createSlice({
  name: 'posts',
  initialState: [],
  reducers: {
    createPost(state, action) {},
    updatePost(state, action) {},
    deletePost(state, action) {}
  }
})

// Extract the action creators object and the reducer
const { actions, reducer } = postsSlice
// Extract and export each action creator by name
export const { createPost, updatePost, deletePost } = actions
// Export the reducer, either as a default or named export
export default reducer
```

### Parameters

`createSlice` accepts a single configuration object parameter, with the following options:

```ts
function createSlice({
    // An object of "case reducers". Key names will be used to generate actions.
    reducers: Object<string, ReducerFunction | ReducerAndPrepareObject>
    // The initial state for the reducer
    initialState: any,
    // A name, used in action types
    name: string,
    // An additional object of "case reducers". Keys should be other action types.
    extraReducers?:
    | Object<string, ReducerFunction>
    | ((builder: ActionReducerMapBuilder<State>) => void)
})
```

### `reducers`

An object containing Redux "case reducer" functions (functions intended to handle a specific action type, equivalent
to a single case statement in a switch).

The keys in the object will be used to generate string action type constants, and these will show up in the Redux
DevTools Extension when they are dispatched. Also, if any other part of the application happens to dispatch an action
with the exact same type string, the corresponding reducer will be run. Therefore, you should give the functions
descriptive names.

This object will be passed to [`createReducer`](./createReducer.md), so the reducers may safely "mutate" the
state they are given.

#### Customizing Generated Action Creators

If you need to customize the creation of the payload value of an action creator by means of a [`prepare callback`](./createAction.md#using-prepare-callbacks-to-customize-action-contents), the value of the appropriate field of the `reducers` argument object should be an object instead of a function. This object must contain two properties: `reducer` and `prepare`. The value of the `reducer` field should be the case reducer function while the value of the `prepare` field should be the prepare callback function.

#### `initialState`

The initial state value for this slice of state.

#### `name`

A string name for this slice of state. Generated action type constants will use this as a prefix.

#### `extraReducers`

One of the key concepts of Redux is that each slice reducer "owns" its slice of state, and that many slice reducers
can independently respond to the same action type. `extraReducers` allows `createSlice` to respond to other action types
besides the types it has generated.

Like `reducers`, `extraReducers` should be an object containing Redux case reducer functions. However, the keys should
be other Redux string action type constants, and `createSlice` will _not_ auto-generate action types or action creators
for reducers included in this parameter.

As with `reducers`, these reducers will also be passed to `createReducer` and may "mutate" their state safely.

If two fields from `reducers` and `extraReducers` happen to end up with the same action type string,
the function from `reducers` will be used to handle that action type.

Action creators that were generated using [`createAction`](./createAction.md) may be used directly as the keys here, using
computed property syntax. (If you are using TypeScript, you may have to use `actionCreator.type` or `actionCreator.toString()`
to force the TS compiler to accept the computed property.)

#### The "builder callback" API for `extraReducers`

Instead of using a simple object as `extraReducers`, you can also use a callback that receives a `ActionReducerMapBuilder` instance.

We recommend using this API if stricter type safety is necessary when defining reducer argument objects.

#### Return Value

`createSlice` will return an object that looks like:

```ts
{
    name : string,
    reducer : ReducerFunction,
    actions : Object<string, ActionCreator>,
}
```



### Examples

```ts
import { createSlice, createAction, PayloadAction } from '@reduxjs/toolkit'
import { createStore, combineReducers } from 'redux'

const incrementBy = createAction<number>('incrementBy')

const counter = createSlice({
  name: 'counter',
  initialState: 0 as number,
  reducers: {
    increment: state => state + 1,
    decrement: state => state - 1,
    multiply: {
      reducer: (state, action: PayloadAction<number>) => state * action.payload,
      prepare: (value: number) => ({ payload: value || 2 }) // fallback if the payload is a falsy value
    }
  },
  // "builder callback API"
  extraReducers: builder =>
    builder.addCase(incrementBy, (state, action) => {
      return state + action.payload
    })
})

const user = createSlice({
  name: 'user',
  initialState: { name: '', age: 20 },
  reducers: {
    setUserName: (state, action) => {
      state.name = action.payload // mutate the state all you want with immer
    }
  },
  // "map object API"
  extraReducers: {
    [counter.actions.increment]: (state, action) => {
      state.age += 1
    }
  }
})

const reducer = combineReducers({
  counter: counter.reducer,
  user: user.reducer
})

const store = createStore(reducer)

store.dispatch(counter.actions.increment())
// -> { counter: 1, user: {name : '', age: 21} }
store.dispatch(counter.actions.increment())
// -> { counter: 2, user: {name: '', age: 22} }
store.dispatch(counter.actions.multiply(3))
// -> { counter: 6, user: {name: '', age: 22} }
store.dispatch(counter.actions.multiply())
// -> { counter: 12, user: {name: '', age: 22} }
console.log(`${counter.actions.decrement}`)
// -> "counter/decrement"
store.dispatch(user.actions.setUserName('eric'))
// -> { counter: 6, user: { name: 'eric', age: 22} }
```
If you encounter this, you may need to restructure your code in a way that avoids the circular references.


## createSelector(...inputSelectors | [inputSelectors], resultFunc)

Takes one or more selectors, or an array of selectors, computes their values and passes them as arguments to `resultFunc`.

`createSelector` determines if the value returned by an input-selector has changed between calls using reference equality (`===`). Inputs to selectors created with `createSelector` should be immutable.

Selectors created with `createSelector` have a cache size of 1. This means they always recalculate when the value of an input-selector changes, as a selector only stores the preceding value of each input-selector.

```js
const mySelector = createSelector(
  state => state.values.value1,
  state => state.values.value2,
  (value1, value2) => value1 + value2
)

// You can also pass an array of selectors
const totalSelector = createSelector(
  [
    state => state.values.value1,
    state => state.values.value2
  ],
  (value1, value2) => value1 + value2
)
```

It can be useful to access the props of a component from within a selector. When a selector is connected to a component with `connect`, the component props are passed as the second argument to the selector:

```js
const abSelector = (state, props) => state.a * props.b

// props only (ignoring state argument)
const cSelector =  (_, props) => props.c

// state only (props argument omitted as not required)
const dSelector = state => state.d

const totalSelector = createSelector(
  abSelector,
  cSelector,
  dSelector,
  (ab, c, d) => ({
    total: ab + c + d
  })
)

```

### `createSelector function` Examples

```js
import { createSelector } from '@reduxjs/toolkit'

const shopItemsSelector = state => state.shop.items
const taxPercentSelector = state => state.shop.taxPercent

const subtotalSelector = createSelector(
  shopItemsSelector,
  items => items.reduce((acc, item) => acc + item.value, 0)
)

const taxSelector = createSelector(
  subtotalSelector,
  taxPercentSelector,
  (subtotal, taxPercent) => subtotal * (taxPercent / 100)
)

export const totalSelector = createSelector(
  subtotalSelector,
  taxSelector,
  (subtotal, tax) => ({ total: subtotal + tax })
)

let exampleState = {
  shop: {
    taxPercent: 8,
    items: [
      { name: 'apple', value: 1.20 },
      { name: 'orange', value: 0.95 },
    ]
  }
}

console.log(subtotalSelector(exampleState)) // 2.15
console.log(taxSelector(exampleState))      // 0.172
console.log(totalSelector(exampleState))    // { total: 2.322 }
```

# React Hooks Overview

## `useState`

```js
const [state, setState] = useState(initialState);
```

Returns a stateful value, and a function to update it.

During the initial render, the returned state (`state`) is the same as the value passed as the first argument (`initialState`).

The `setState` function is used to update the state. It accepts a new state value and enqueues a re-render of the component.

```js
setState(newState);
```

During subsequent re-renders, the first value returned by `useState` will always be the most recent state after applying updates.

>Note
>
>React guarantees that `setState` function identity is stable and won't change on re-renders. This is why it's safe to omit from the `useEffect` or `useCallback` dependency list.

#### Functional updates

If the new state is computed using the previous state, you can pass a function to `setState`. The function will receive the previous value, and return an updated value. Here's an example of a counter component that uses both forms of `setState`:

```js
function Counter({initialCount}) {
  const [count, setCount] = useState(initialCount);
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(initialCount)}>Reset</button>
      <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
      <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
    </>
  );
}
```

The "+" and "-" buttons use the functional form, because the updated value is based on the previous value. But the "Reset" button uses the normal form, because it always sets the count back to the initial value.

If your update function returns the exact same value, the subsequent rerender will be skipped completely.

> Note
>
> Unlike the `setState` method found in class components, `useState` does not automatically merge update objects. You can replicate this behavior by combining the function updater form with object spread syntax:
>
> ```js
> setState(prevState => {
>   // Object.assign would also work
>   return {...prevState, ...updatedValues};
> });
> ```
>

### Equivalent Class Example

If you used classes in React before, this code should look familiar:

```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

### Declaring a State Variable

In a class, we initialize the `count` state to `0` by setting `this.state` to `{ count: 0 }` in the constructor:

```js{4-6}
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
```

In a function component, we have no `this`, so we can't assign or read `this.state`. Instead, we call the `useState` Hook directly inside our component:

```js{4,5}
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);
```

**What does calling `useState` do?** It declares a "state variable". Our variable is called `count` but we could call it anything else, like `banana`. This is a way to "preserve" some values between the function calls â€” `useState` is a new way to use the exact same capabilities that `this.state` provides in a class. Normally, variables "disappear" when the function exits but state variables are preserved by React.

**What do we pass to `useState` as an argument?** The only argument to the `useState()` Hook is the initial state. Unlike with classes, the state doesn't have to be an object. We can keep a number or a string if that's all we need. In our example, we just want a number for how many times the user clicked, so pass `0` as initial state for our variable. (If we wanted to store two different values in state, we would call `useState()` twice.)

**What does `useState` return?** It returns a pair of values: the current state and a function that updates it. This is why we write `const [count, setCount] = useState()`. This is similar to `this.state.count` and `this.setState` in a class, except you get them in a pair. If you're not familiar with the syntax we used, we'll come back to it [at the bottom of this page](/docs/hooks-state.html#tip-what-do-square-brackets-mean).

Now that we know what the `useState` Hook does, our example should make more sense:

```js{4,5}
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);
```

We declare a state variable called `count`, and set it to `0`. React will remember its current value between re-renders, and provide the most recent one to our function. If we want to update the current `count`, we can call `setCount`.

>Note
>
>You might be wondering: why is `useState` not named `createState` instead?
>
>"Create" wouldn't be quite accurate because the state is only created the first time our component renders. During the next renders, `useState` gives us the current state. Otherwise it wouldn't be "state" at all! There's also a reason why Hook names *always* start with `use`. We'll learn why later in the [Rules of Hooks](/docs/hooks-rules.html).

### Reading State

When we want to display the current count in a class, we read `this.state.count`:

```js
  <p>You clicked {this.state.count} times</p>
```

In a function, we can use `count` directly:


```js
  <p>You clicked {count} times</p>
```

### Updating State

In a class, we need to call `this.setState()` to update the `count` state:

```js{1}
  <button onClick={() => this.setState({ count: this.state.count + 1 })}>
    Click me
  </button>
```

In a function, we already have `setCount` and `count` as variables so we don't need `this`:

```js{1}
  <button onClick={() => setCount(count + 1)}>
    Click me
  </button>
```


### Tip: What Do Square Brackets Mean?

You might have noticed the square brackets when we declare a state variable:

```js
  const [count, setCount] = useState(0);
```

The names on the left aren't a part of the React API. You can name your own state variables:

```js
  const [fruit, setFruit] = useState('banana');
```

This JavaScript syntax is called ["array destructuring"](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Array_destructuring). It means that we're making two new variables `fruit` and `setFruit`, where `fruit` is set to the first value returned by `useState`, and `setFruit` is the second. It is equivalent to this code:

```js
  var fruitStateVariable = useState('banana'); // Returns a pair
  var fruit = fruitStateVariable[0]; // First item in a pair
  var setFruit = fruitStateVariable[1]; // Second item in a pair
```

When we declare a state variable with `useState`, it returns a pair â€” an array with two items. The first item is the current value, and the second is a function that lets us update it. Using `[0]` and `[1]` to access them is a bit confusing because they have a specific meaning. This is why we use array destructuring instead.

>Note
>
>You might be curious how React knows which component `useState` corresponds to since we're not passing anything like `this` back to React. We'll answer [this question](/docs/hooks-faq.html#how-does-react-associate-hook-calls-with-components) and many others in the FAQ section.

### Tip: Using Multiple State Variables

Declaring state variables as a pair of `[something, setSomething]` is also handy because it lets us give *different* names to different state variables if we want to use more than one:

```js
function ExampleWithManyStates() {
  // Declare multiple state variables!
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
```

In the above component, we have `age`, `fruit`, and `todos` as local variables, and we can update them individually:

```js
  function handleOrangeClick() {
    // Similar to this.setState({ fruit: 'orange' })
    setFruit('orange');
  }
```

## `useEffect`

```js
useEffect(didUpdate);
```

Accepts a function that contains imperative, possibly effectful code.

Mutations, subscriptions, timers, logging, and other side effects are not allowed inside the main body of a function component (referred to as React's _render phase_). Doing so will lead to confusing bugs and inconsistencies in the UI.

Instead, use `useEffect`. The function passed to `useEffect` will run after the render is committed to the screen. Think of effects as an escape hatch from React's purely functional world into the imperative world.

By default, effects run after every completed render, but you can choose to fire them [only when certain values have changed](#conditionally-firing-an-effect).

### Cleaning up an effect

Often, effects create resources that need to be cleaned up before the component leaves the screen, such as a subscription or timer ID. To do this, the function passed to `useEffect` may return a clean-up function. For example, to create a subscription:

```js
useEffect(() => {
  const subscription = props.source.subscribe();
  return () => {
    // Clean up the subscription
    subscription.unsubscribe();
  };
});
```

The clean-up function runs before the component is removed from the UI to prevent memory leaks. Additionally, if a component renders multiple times (as they typically do), the **previous effect is cleaned up before executing the next effect**. In our example, this means a new subscription is created on every update. To avoid firing an effect on every update, refer to the next section.

### Timing of effects

Unlike `componentDidMount` and `componentDidUpdate`, the function passed to `useEffect` fires **after** layout and paint, during a deferred event. This makes it suitable for the many common side effects, like setting up subscriptions and event handlers, because most types of work shouldn't block the browser from updating the screen.

However, not all effects can be deferred. For example, a DOM mutation that is visible to the user must fire synchronously before the next paint so that the user does not perceive a visual inconsistency. (The distinction is conceptually similar to passive versus active event listeners.) For these types of effects, React provides one additional Hook called [`useLayoutEffect`](#uselayouteffect). It has the same signature as `useEffect`, and only differs in when it is fired.

Although `useEffect` is deferred until after the browser has painted, it's guaranteed to fire before any new renders. React will always flush a previous render's effects before starting a new update.

### Conditionally firing an effect

The default behavior for effects is to fire the effect after every completed render. That way an effect is always recreated if one of its dependencies changes.

However, this may be overkill in some cases, like the subscription example from the previous section. We don't need to create a new subscription on every update, only if the `source` prop has changed.

To implement this, pass a second argument to `useEffect` that is the array of values that the effect depends on. Our updated example now looks like this:

```js
useEffect(
  () => {
    const subscription = props.source.subscribe();
    return () => {
      subscription.unsubscribe();
    };
  },
  [props.source],
);
```

Now the subscription will only be recreated when `props.source` changes.

>Note
>
>If you use this optimization, make sure the array includes **all values from the component scope (such as props and state) that change over time and that are used by the effect**. Otherwise, your code will reference stale values from previous renders. Learn more about [how to deal with functions](/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) and what to do when the [array values change too often](/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often).
>
>If you want to run an effect and clean it up only once (on mount and unmount), you can pass an empty array (`[]`) as a second argument. This tells React that your effect doesn't depend on *any* values from props or state, so it never needs to re-run. This isn't handled as a special case -- it follows directly from how the dependencies array always works.
>
>If you pass an empty array (`[]`), the props and state inside the effect will always have their initial values. While passing `[]` as the second argument is closer to the familiar `componentDidMount` and `componentWillUnmount` mental model, there are usually [better](/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) [solutions](/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often) to avoid re-running effects too often. Also, don't forget that React defers running `useEffect` until after the browser has painted, so doing extra work is less of a problem.
>
>
>We recommend using the [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) rule as part of our [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation) package. It warns when dependencies are specified incorrectly and suggests a fix.

The array of dependencies is not passed as arguments to the effect function. Conceptually, though, that's what they represent: every value referenced inside the effect function should also appear in the dependencies array. In the future, a sufficiently advanced compiler could create this array automatically.

## `useCallback`

```js
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

Returns a [memoized](https://en.wikipedia.org/wiki/Memoization) callback.

Pass an inline callback and an array of dependencies. `useCallback` will return a memoized version of the callback that only changes if one of the dependencies has changed. This is useful when passing callbacks to optimized child components that rely on reference equality to prevent unnecessary renders (e.g. `shouldComponentUpdate`).

`useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`.

> Note
>
> The array of dependencies is not passed as arguments to the callback. Conceptually, though, that's what they represent: every value referenced inside the callback should also appear in the dependencies array. In the future, a sufficiently advanced compiler could create this array automatically.
>
> We recommend using the [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) rule as part of our [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation) package. It warns when dependencies are specified incorrectly and suggests a fix.


## `useMemo`

```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

Returns a [memoized](https://en.wikipedia.org/wiki/Memoization) value.

Pass a "create" function and an array of dependencies. `useMemo` will only recompute the memoized value when one of the dependencies has changed. This optimization helps to avoid expensive calculations on every render.

Remember that the function passed to `useMemo` runs during rendering. Don't do anything there that you wouldn't normally do while rendering. For example, side effects belong in `useEffect`, not `useMemo`.

If no array is provided, a new value will be computed on every render.

**You may rely on `useMemo` as a performance optimization, not as a semantic guarantee.** In the future, React may choose to "forget" some previously memoized values and recalculate them on next render, e.g. to free memory for offscreen components. Write your code so that it still works without `useMemo` â€” and then add it to optimize performance.

> Note
>
> The array of dependencies is not passed as arguments to the function. Conceptually, though, that's what they represent: every value referenced inside the function should also appear in the dependencies array. In the future, a sufficiently advanced compiler could create this array automatically.
>
> We recommend using the [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) rule as part of our [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation) package. It warns when dependencies are specified incorrectly and suggests a fix.

## `useRef`

```js
const refContainer = useRef(initialValue);
```

`useRef` returns a mutable ref object whose `.current` property is initialized to the passed argument (`initialValue`). The returned object will persist for the full lifetime of the component.

A common use case is to access a child imperatively:

```js
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` points to the mounted text input element
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

Essentially, `useRef` is like a "box" that can hold a mutable value in its `.current` property.

You might be familiar with refs primarily as a way to [access the DOM](/docs/refs-and-the-dom.html). If you pass a ref object to React with `<div ref={myRef} />`, React will set its `.current` property to the corresponding DOM node whenever that node changes.

However, `useRef()` is useful for more than the `ref` attribute. It's [handy for keeping any mutable value around](/docs/hooks-faq.html#is-there-something-like-instance-variables) similar to how you'd use instance fields in classes.

This works because `useRef()` creates a plain JavaScript object. The only difference between `useRef()` and creating a `{current: ...}` object yourself is that `useRef` will give you the same ref object on every render.

Keep in mind that `useRef` *doesn't* notify you when its content changes. Mutating the `.current` property doesn't cause a re-render. If you want to run some code when React attaches or detaches a ref to a DOM node, you may want to use a [callback ref](/docs/hooks-faq.html#how-can-i-measure-a-dom-node) instead.

## `useSelector()`

```js
const result: any = useSelector(selector: Function, equalityFn?: Function)
```

Allows you to extract data from the Redux store state, using a selector function.

> **Note**: The selector function should be [pure](https://en.wikipedia.org/wiki/Pure_function) since it is potentially executed multiple times and at arbitrary points in time.

The selector is approximately equivalent to the [`mapStateToProps` argument to `connect`](../using-react-redux/connect-extracting-data-with-mapStateToProps.md) conceptually. The selector will be called with the entire Redux store state as its only argument. The selector will be run whenever the function component renders. `useSelector()` will also subscribe to the Redux store, and run your selector whenever an action is dispatched.

However, there are some differences between the selectors passed to `useSelector()` and a `mapState` function:

- The selector may return any value as a result, not just an object. The return value of the selector will be used as the return value of the `useSelector()` hook.
- When an action is dispatched, `useSelector()` will do a reference comparison of the previous selector result value and the current result value. If they are different, the component will be forced to re-render. If they are the same, the component will not re-render.
- The selector function does _not_ receive an `ownProps` argument. However, props can be used through closure (see the examples below) or by using a curried selector.
- Extra care must be taken when using memoizing selectors (see examples below for more details).
- `useSelector()` uses strict `===` reference equality checks by default, not shallow equality (see the following section for more details).

> **Note**: There are potential edge cases with using props in selectors that may cause errors. See the [Usage Warnings](#usage-warnings) section of this page for further details.

You may call `useSelector()` multiple times within a single function component. Each call to `useSelector()` creates an individual subscription to the Redux store. Because of the React update batching behavior used in React Redux v7, a dispatched action that causes multiple `useSelector()`s in the same component to return new values _should_ only result in a single re-render.

### Equality Comparisons and Updates

When the function component renders, the provided selector function will be called and its result will be returned
from the `useSelector()` hook. (A cached result may be returned if the selector has been run and hasn't changed.)

However, when an action is dispatched to the Redux store, `useSelector()` only forces a re-render if the selector result
appears to be different than the last result. As of v7.1.0-alpha.5, the default comparison is a strict `===` reference
comparison. This is different than `connect()`, which uses shallow equality checks on the results of `mapState` calls
to determine if re-rendering is needed. This has several implications on how you should use `useSelector()`.

With `mapState`, all individual fields were returned in a combined object. It didn't matter if the return object was
a new reference or not - `connect()` just compared the individual fields. With `useSelector()`, returning a new object
every time will _always_ force a re-render by default. If you want to retrieve multiple values from the store, you can:

- Call `useSelector()` multiple times, with each call returning a single field value
- Use Reselect or a similar library to create a memoized selector that returns multiple values in one object, but
  only returns a new object when one of the values has changed.
- Use the `shallowEqual` function from React-Redux as the `equalityFn` argument to `useSelector()`, like:

```js
import { shallowEqual, useSelector } from 'react-redux'

// later
const selectedData = useSelector(selectorReturningObject, shallowEqual)
```

The optional comparison function also enables using something like Lodash's `_.isEqual()` or Immutable.js's comparison capabilities.

### `useSelector` Examples

Basic usage:

```jsx
import React from 'react'
import { useSelector } from 'react-redux'

export const CounterComponent = () => {
  const counter = useSelector(state => state.counter)
  return <div>{counter}</div>
}
```

Using props via closure to determine what to extract:

```jsx
import React from 'react'
import { useSelector } from 'react-redux'

export const TodoListItem = props => {
  const todo = useSelector(state => state.todos[props.id])
  return <div>{todo.text}</div>
}
```

#### Using memoizing selectors

When using `useSelector` with an inline selector as shown above, a new instance of the selector is created whenever the component is rendered. This works as long as the selector does not maintain any state. However, memoizing selectors (e.g. created via `createSelector` from `reselect`) do have internal state, and therefore care must be taken when using them. Below you can find typical usage scenarios for memoizing selectors.

When the selector does only depend on the state, simply ensure that it is declared outside of the component so that the same selector instance is used for each render:

```jsx
import React from 'react'
import { useSelector } from 'react-redux'
import { createSelector } from 'reselect'

const selectNumOfDoneTodos = createSelector(
  state => state.todos,
  todos => todos.filter(todo => todo.isDone).length
)

export const DoneTodosCounter = () => {
  const NumOfDoneTodos = useSelector(selectNumOfDoneTodos)
  return <div>{NumOfDoneTodos}</div>
}

export const App = () => {
  return (
    <>
      <span>Number of done todos:</span>
      <DoneTodosCounter />
    </>
  )
}
```

The same is true if the selector depends on the component's props, but will only ever be used in a single instance of a single component:

```jsx
import React from 'react'
import { useSelector } from 'react-redux'
import { createSelector } from 'reselect'

const selectNumOfTodosWithIsDoneValue = createSelector(
  state => state.todos,
  (_, isDone) => isDone,
  (todos, isDone) => todos.filter(todo => todo.isDone === isDone).length
)

export const TodoCounterForIsDoneValue = ({ isDone }) => {
  const NumOfTodosWithIsDoneValue = useSelector(state =>
    selectNumOfTodosWithIsDoneValue(state, isDone)
  )

  return <div>{NumOfTodosWithIsDoneValue}</div>
}

export const App = () => {
  return (
    <>
      <span>Number of done todos:</span>
      <TodoCounterForIsDoneValue isDone={true} />
    </>
  )
}
```

However, when the selector is used in multiple component instances and depends on the component's props, you need to ensure that each component instance gets its own selector instance (see [here](https://github.com/reduxjs/reselect#accessing-react-props-in-selectors) for a more thorough explanation of why this is necessary):

```jsx
import React, { useMemo } from 'react'
import { useSelector } from 'react-redux'
import { createSelector } from 'reselect'

const makeNumOfTodosWithIsDoneSelector = () =>
  createSelector(
    state => state.todos,
    (_, isDone) => isDone,
    (todos, isDone) => todos.filter(todo => todo.isDone === isDone).length
  )

export const TodoCounterForIsDoneValue = ({ isDone }) => {
  const selectNumOfTodosWithIsDone = useMemo(
    makeNumOfTodosWithIsDoneSelector,
    []
  )

  const numOfTodosWithIsDoneValue = useSelector(state =>
    selectNumOfTodosWithIsDone(state, isDone)
  )

  return <div>{numOfTodosWithIsDoneValue}</div>
}

export const App = () => {
  return (
    <>
      <span>Number of done todos:</span>
      <TodoCounterForIsDoneValue isDone={true} />
      <span>Number of unfinished todos:</span>
      <TodoCounterForIsDoneValue isDone={false} />
    </>
  )
}
```


## `useDispatch()`

```js
const dispatch = useDispatch()
```

This hook returns a reference to the `dispatch` function from the Redux store. You may use it to dispatch actions as needed.

#### Examples

```jsx
import React from 'react'
import { useDispatch } from 'react-redux'

export const CounterComponent = ({ value }) => {
  const dispatch = useDispatch()

  return (
    <div>
      <span>{value}</span>
      <button onClick={() => dispatch({ type: 'increment-counter' })}>
        Increment counter
      </button>
    </div>
  )
}
```

When passing a callback using `dispatch` to a child component, it is recommended to memoize it with `useCallback`, since otherwise child components may render unnecessarily due to the changed reference.

```jsx
import React, { useCallback } from 'react'
import { useDispatch } from 'react-redux'

export const CounterComponent = ({ value }) => {
  const dispatch = useDispatch()
  const incrementCounter = useCallback(
    () => dispatch({ type: 'increment-counter' }),
    [dispatch]
  )

  return (
    <div>
      <span>{value}</span>
      <MyIncrementButton onIncrement={incrementCounter} />
    </div>
  )
}

export const MyIncrementButton = React.memo(({ onIncrement }) => (
  <button onClick={onIncrement}>Increment counter</button>
))
```


## `Hooks Rules`

### Only Call Hooks at the Top Level

**Don't call Hooks inside loops, conditions, or nested functions.** Instead, always use Hooks at the top level of your React function. By following this rule, you ensure that Hooks are called in the same order each time a component renders. That's what allows React to correctly preserve the state of Hooks between multiple `useState` and `useEffect` calls. (If you're curious, we'll explain this in depth [below](#explanation).)

### Only Call Hooks from React Functions

**Don't call Hooks from regular JavaScript functions.** Instead, you can:

* ✅ Call Hooks from React function components.
* ✅ Call Hooks from custom Hooks (we'll learn about them [on the next page](/docs/hooks-custom.html)).

By following this rule, you ensure that all stateful logic in a component is clearly visible from its source code.

### ESLint Plugin

We released an ESLint plugin called [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks) that enforces these two rules. You can add this plugin to your project if you'd like to try it:

```bash
npm install eslint-plugin-react-hooks --save-dev
```

```js
// Your ESLint configuration
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error", // Checks rules of Hooks
    "react-hooks/exhaustive-deps": "warn" // Checks effect dependencies
  }
}
```

This plugin is included by default in [Create React App](/docs/create-a-new-react-app.html#create-react-app).

**You can skip to the next page explaining how to write [your own Hooks](/docs/hooks-custom.html) now.** On this page, we'll continue by explaining the reasoning behind these rules.

### Explanation

As we [learned earlier](/docs/hooks-state.html#tip-using-multiple-state-variables), we can use multiple State or Effect Hooks in a single component:

```js
function Form() {
  // 1. Use the name state variable
  const [name, setName] = useState('Mary');

  // 2. Use an effect for persisting the form
  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });

  // 3. Use the surname state variable
  const [surname, setSurname] = useState('Poppins');

  // 4. Use an effect for updating the title
  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });

  // ...
}
```

So how does React know which state corresponds to which `useState` call? The answer is that **React relies on the order in which Hooks are called**. Our example works because the order of the Hook calls is the same on every render:

```js
// ------------
// First render
// ------------
useState('Mary')           // 1. Initialize the name state variable with 'Mary'
useEffect(persistForm)     // 2. Add an effect for persisting the form
useState('Poppins')        // 3. Initialize the surname state variable with 'Poppins'
useEffect(updateTitle)     // 4. Add an effect for updating the title

// -------------
// Second render
// -------------
useState('Mary')           // 1. Read the name state variable (argument is ignored)
useEffect(persistForm)     // 2. Replace the effect for persisting the form
useState('Poppins')        // 3. Read the surname state variable (argument is ignored)
useEffect(updateTitle)     // 4. Replace the effect for updating the title

// ...
```

As long as the order of the Hook calls is the same between renders, React can associate some local state with each of them. But what happens if we put a Hook call (for example, the `persistForm` effect) inside a condition?

```js
  // 🔴 We're breaking the first rule by using a Hook in a condition
  if (name !== '') {
    useEffect(function persistForm() {
      localStorage.setItem('formData', name);
    });
  }
```

The `name !== ''` condition is `true` on the first render, so we run this Hook. However, on the next render the user might clear the form, making the condition `false`. Now that we skip this Hook during rendering, the order of the Hook calls becomes different:

```js
useState('Mary')           // 1. Read the name state variable (argument is ignored)
// useEffect(persistForm)  // 🔴 This Hook was skipped!
useState('Poppins')        // 🔴 2 (but was 3). Fail to read the surname state variable
useEffect(updateTitle)     // 🔴 3 (but was 4). Fail to replace the effect
```

React wouldn't know what to return for the second `useState` Hook call. React expected that the second Hook call in this component corresponds to the `persistForm` effect, just like during the previous render, but it doesn't anymore. From that point, every next Hook call after the one we skipped would also shift by one, leading to bugs.

**This is why Hooks must be called on the top level of our components.** If we want to run an effect conditionally, we can put that condition *inside* our Hook:

```js
  useEffect(function persistForm() {
    // 👍 We're not breaking the first rule anymore
    if (name !== '') {
      localStorage.setItem('formData', name);
    }
  });
```

**Note that you don't need to worry about this problem if you use the [provided lint rule](https://www.npmjs.com/package/eslint-plugin-react-hooks).** But now you also know *why* Hooks work this way, and which issues the rule is preventing.