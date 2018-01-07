# redux-object-format

A recommendation for combining elements of a [redux](https://github.com/reactjs/redux) workflow (actions, reducers, action creators, selectors, etc.) into a single object.

Inspired by [Ducks](https://github.com/erikras/ducks-modular-redux) (and somewhat by [Ducks++](https://medium.com/@DjamelH/ducks-redux-reducer-bundles-44267f080d22)), and with a tip of the hat to [extensible-duck et al.](https://github.com/investtools/extensible-duck). [Ducks](https://github.com/erikras/ducks-modular-redux) does an excellent job of recognizing and solving the issue of related code being kept in separate files. This is an approach that builds on that idea by packing the related elements into a more compact and organized format.

## Motivation

* To define a guideline for bundling related elements in a redux store into a single object.
* Provide greater ease of importing, better organization and easier automation of creation.
* Respect the wide variety of projects in development in the real world and try to avoid any unnecessary rules.

## Format

All related elements in a redux flow can be combined within a single object. This is similar to the module format described by [Ducks](https://github.com/erikras/ducks-modular-redux) but one a major difference. With Ducks you export each property separately with the reducer as the default export. With redux-object-format, the entire contents of the "duck" is grouped together for export and individual pieces are optional.

The format is an object containing several properties. Each property is optional (an object with just a reducer and nothing else is valid).

* reduxObject

  * `actions` - An object with [Action Types](https://redux.js.org/docs/basics/Actions.html#actions). Each key is a string action type constant, each value is an action type name. e.g. `{ SET_FIRST_NAME: "myProject/name/SET_FIRST_NAME"}`
  * `creators` - An object with [Action Creators](https://redux.js.org/docs/basics/Actions.html#action-creators). Each key is the function name, each value is a function. e.g. `{ setFirstName: name => { type: "SET_FIRST_NAME", payload: name }}`
  * `reducer` - The [Reducer](https://redux.js.org/docs/basics/Reducers.html#reducers) function for this object.
  * `reducers` - An object with mappings from action types to reducer functions that handle only that type of action. The intent would be to [combine](https://redux.js.org/docs/api/combineReducers.html) these single-purpose reducers into one that handles all action types. This also allows easy modifications to the list of action types accepted by the reducer.
  * `selector` - The [selector](https://redux.js.org/docs/recipes/ComputingDerivedData.html) function to get the value from the main state tree.
  * `selectors` - An object with mappings from selector names to selector functions. Use this for any additional selectors. The root selector should be included in these.
  * `sagas` - When using [redux-saga](https://github.com/redux-saga/redux-saga)s, this is an object with mappings from saga name to saga functions.
  * `meta` - An object containing additional metadata about the bundle.

    * `name` - A string for the name of this bundle.
    * `mountPoint` - A string that locates the value in the main redux store's state tree. This would usually be the same value as "name" but could be a nested path to the value. e.g. `user.contact.firstName`
    * Any additional metadata could be added here.

### Recommendations for modules

When exporting a bundled object, the module...

* MUST export default a redux object as described above.
* SHOULD export groups of values using the same name as above.
* MAY export any individual values as long as it is clear what the purpose of those values are.

A trivial example of how this might look:

```
// export this individual action type
export const SET_NAME = "SET_NAME";

// export groups of values
export const actions = { SET_NAME };
export const creators = {
  setName: name => ({ type: SET_NAME, payload: name }),
};
export const reducers = { SET_NAME: value => value };
export const reducer = (state, action) => {
  if (action.type === SET_NAME) return reducers.SET_NAME;
};

export const meta = { name: "name", mountPoint: "name" };
export const selector = state => state[meta.mountPoint];
export const selectors = {
  getName: selector,
  getFirstName: state => selector(state).split(" ")[0],
  getLastName: state => selector(state).split(" ")[1],
};

// Export redux object
export default { meta, actions, creators, reducers, reducer, selectors, selector }
```

### Naming recommendations

In the interest of not being overly-prescriptive, this spec doesn't define rules for naming the elements within the bundle.

## FAQ

**What about [Thunks](https://github.com/gaearon/redux-thunk)?**

Thunks function the same way as action creators so would go in `creators`. If you disagree, please open an issue explaining why.

**Isn't it a bad idea to group actions with a specific reducer? Actions are supposed to be usable by multiple reducers.**

It's a bad idea to treat actions as though only one reducer can use them, but I don't think it's necessarily bad to group them with a reducer. They can still be imported by another reducer.

However, this shouldn't be an issue with this format since all of the groups of elements are optional. That is, you can create a bundle with only `actions` or one with a `reducer` but no `actions`.

```
import {actions} from "redux/name";

const placeholderReducer = (state, action) => {
  if (action.type === SET_NAME) {
    if (name = "") {
      return "Please enter your name";
    }
  }
};

export default { reducer: placeholderReducer };
```
