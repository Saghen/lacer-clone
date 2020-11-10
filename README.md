<p align="center">
  <img src="https://i.imgur.com/ecqYJwC.png" width="80%">
</p>

# Lacer
<p align="center">
  <img src="https://img.shields.io/static/v1?label=build&message=passing&color=success&style=flat-square">
  <img src="https://img.shields.io/static/v1?label=version&message=0.0.1-rc4&color=blue&style=flat-square">
</p>

Very simple and powerful state management solution for any application built on Laco and Immer.

Set up your stores and subscribe to them. Easy as that!

`npm install lacer`

## Summary
- :rocket: Simple to use
- :tada: Lightweight (<2kbs gzipped)
- :sparkles: Partial [Redux DevTools Extension](https://github.com/zalmoxisus/redux-devtools-extension) support (time travel thanks to Laco)

## Example
```javascript
import { Store } from 'lacer'

// Creating a new store with an initial state { count: 0 }
const CounterStore = new Store({ count: 0 })

// Implementing some actions to update the store
const increment = () => CounterStore.set((state) => state.count++)
const decrement = () => CounterStore.set((state) => state.count--)

increment()
expect(CounterStore.get().count).toBe(1) // Success

decrement()
expect(CounterStore.get().count).toBe(0) // Success
```

## Redux DevTools Extension
Check out [Redux DevTools Extension](https://github.com/zalmoxisus/redux-devtools-extension).
### Time travel
Just click on the stopwatch icon and you will get a slider which you can play with.
That's it! :)

## React Native Debugger
Check out [React Native Debugger](https://github.com/jhen0409/react-native-debugger).
### Time travel
Works as you would expect :)!

## API
### `Store<T>(initialState: T, name?: string)`
```typescript
// Initializing a new store with an initial state and a name:
interface INewStore = {
  count: number
}

const NewStore = Store<INewStore>({ count: 0 }, "Counter")
```
The name is optional and is used to get an overview of action and store relationship in Redux DevTools Extension. Action names for the Store will now show up as `Counter - ${actionType}` in DevTools Extension where as before only `${actionType}` was shown.

### `Store.get(): T`
```javascript
// Getting the state of the store
Store.get()
```
Returns an object which could be something like `{ count: 0 }` following the example.

### `Store.set(state: SetStateFunc, info?: string)`
`type SetStateFunc<T> = (state: Draft<T> => void)`
Technically, this function does allow any value to be returned to allow patterns similar to what you'll see in the example where the assignment is returned. **However, the return value is ignored.**

T inherits from the generic specified during the initialization of the Store. The SetStateFunc provides a `Draft<T>` from Immer. For more information on how to use Immer, [check it out here](https://immerjs.github.io/immer/docs/introduction).
```javascript
// Setting a new state and passing an optional action name "increment"
Store.set((state) => (state.count++), "increment")
```

### `Store.replace(state: ReplaceStateFunc<, info?: String)`
`type ReplaceStateFunc<T> = (state: T) => T`

T inherits from the generic specified during the initialization of the Store. This function allows you to completely replace the object used in the store and fires all subscription handlers afterwards regardless of the properties they listen to.
```javascript
// Setting a new state and passing an optional action name "increment"
Store.replace((state) => { /* return modified state */}, "increment")
```

### `Store.addMiddleware(middleware: MiddlewareFunc)`
`type MiddlewareFunc<T> = (state: T, draft: Draft<T>, actionType?: string) => boolean | undefined`

A middleware will intercept a state change (set or replace) before it is made. If the middleware returns false, the change is discarded. If it returns true or undefined, the next middleware will run. Setting values is identical to `Store.set` since it uses immer internally as well. For setting the state, use `draft`. For getting the state, use `state`.

Note: You can mutate this object freely since it is generated by Immer.
```javascript
// Setting a condition to prevent count from going below 0
// and a special case for `SudoDecrement` action which CAN make count go below 0
CounterStore.addMiddleware((state, _, actiontype) => state.count >= 0 || actionType === 'SudoDecrement')
```

### `Store.removeMiddleware(middleware: MiddlewareFunc)`
`type MiddlewareFunc<T> = (state: T, draft: Draft<T>, actionType?: string) => boolean | undefined`

The remove middleware function takes a middleware that matches a previously added middleware and removes it.
```javascript
// Setting a condition to prevent count from going below 0
// and a special case for `SudoDecrement` action which CAN make count go below 0
const myMiddleware = (state) => (state.count++)
CounterStore.addMiddleware(myMiddleware)

CounterStore.set((state) => (state.count = 0))
expect(CounterStore.get().count).toBe(1) // Success

CounterStore.removeMiddleware(myMiddleware)
CounterStore.set((state) => (state.count = 0))
expect(CounterStore.get().count).toBe(0) // Success
```

### `Store.subscribe(listener: ListenerFunc, properties?: string[]): ListenerUnsubscribeFunc`
`type ListenerFunc<T> = (state: T, oldState: T, changes?: string[]) => void`

A subscriber will fire whenever its properties match the properties changed during a set. It will always fire when a replace or reset is called. If properties is undefined, it will always fire on state change.

Note: DO NOT attempt to mutate the state object as it will throw an error.
```javascript
// Setting a condition to prevent count from going below 0
// and a special case for `SudoDecrement` action which CAN make count go below 0
const unsubscribe = CounterStore.subscribe(
  (state, oldState) => console.log(state.count, state.oldCount),
  ['count']
)
CounterStore.set((state) => (state.count++)) // "1 0" printed to console

CounterStore.set((state) => (state.someOtherProperty++)) // Nothing printed to console

unsubscribe()
CounterStore.set((state) => (state.count++)) // Nothing printed to console
```

### `Store.unsubscribe(listener: ListenerFunc)`
`type ListenerFunc<T> = (state: T, oldState: T, changes?: string[]) => void`

The unsubscribe function takes a listener function that matches a previously added subscription and removes it.
```javascript
// Setting a condition to prevent count from going below 0
// and a special case for `SudoDecrement` action which CAN make count go below 0
const myListener = (state) => (console.log(state.count))

CounterStore.subscribe(myListener)
CounterStore.set((state) => (state.count++)) // "1" printed to console

CounterStore.unsubscribe(myListener)
CounterStore.set((state) => (state.count++)) // Nothing printed to console
```

### `Store.reset(force: boolean): boolean`
```javascript
// Resets the store to initial state
Store.reset()
```
Reset will bring back the original state of the Store when it was initialized. This is stored in `Store.initialState`. It will return true on success and false on failure. If force is true, it will run all middleware regardless of their success and will always reset the store.

### `Store.dispatch(value: any, info: String)`
```javascript
// Dispatching an action that does not change the state of the store
Store.dispatch(changeLocation(), "Location change")
```
You might want to dispatch an action that is associated with a certain store but don't want to change the state. The action will in this case be shown as `StoreName - Location change`.

### `dispatch(value: any, info: String)`
```javascript
import { dispatch } from 'lacer'

// Dispatching a global action that does not change any state
dispatch(changeLocation(), "Location change")
```
You might want to dispatch a global action that is **NOT** associated with any store. The action will in this case just be shown as `Location change`.

## Testing
Testing using [jest](https://github.com/facebook/jest):
```javascript
import { ICounterState } from '/types'
import { Store } from 'lacer'

test('CounterStore simple actions', () => {
  // Creating a new store with an initial state { count: 0 }
  const CounterStore = new Store<ICounterState>({ count: 0 }, 'Counter')

  // Implementing an action to update the store
  const increment = () => CounterStore.set((prev) => prev.count++, 'Increment')

  expect(CounterStore.get().count).toBe(0)

  increment()
  expect(CounterStore.get().count).toBe(1)
})
```

## Credits
Based on:
- [Laco](https://github.com/deamme/laco)

Depends on:
- [Immer](https://github.com/immerjs/immer)

Special Thanks
- [Austin McCalley](https://github.com/austinmccalley) for the name
