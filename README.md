# Kepler.gl

Kepler.gl is the core package of Voyager. It is a map visualization tool that provides the UI
to visualize any type of geospatial dataset. For what it is capable of. Take a look at [Voyager](voyager.uberinternal.com)

Kepler.gl is a redux component that uses redux reducer to store and manage state transitions.
The package consists of a reducer and the UI component to render and customize the map

##Links

### Node
Use Node v6 and above, older node version has not been tested

### Install
```
npm install --save @uber/kepler.gl
```

### Local dev
```
npm install
```
or
```
yarn
```
then
```
npm start
```

An Example app will be served at
http://localhost:8080/

### Basic Usage
You can also take a look at `kepler.gl/examples/simple-app` for How to uer kepler.gl in your app
Here are the basic steps:

#### 1. Mount kepler.gl reducer in your app reducer. Kepler.gl is using [react-palm](https://github.com/btford/react-palm) to handle side effects.
You need to add `taskMiddleware` to your store too. We are actively working on a solution where
`react-palm` will not be required. However it is still a very nice side effects management tool that much easy for testing (unlike react-thunk).

```
import {keplerGlReducer} from '@uber/kepler.gl';
import {createStore, combineReducers, applyMiddleware, compose} from 'redux';
import {taskMiddleware} from 'react-palm';

const reducers = combineReducers({
  // <-- mount kepler.gl reducer in your app
  keplerGl: keplerGlReducer,

  // Your other reducers here
  app: appReducer
});

// using createStore
const store = createStore(reducer, applyMiddleWare(taskMiddleware))

// using enhancers
const initialState = {}
const middlewares = [taskMiddleware]
const enhancers = [
  applyMiddleware(...middlewares)
]

const store = createStore(reducer, initialState, compose(...enhancers))
```

If you mount kepler.gl reducer in another address instead of `keplerGl`, or kepler.gl reducer is not
mounted at root of your state, you will need to specify the path to it when you mount the component
with the `getState` prop.

#### 2. Mount kepler.gl Component

```
import KeplerGl from '@uber/kepler.gl';

const Map = props => (
  <KeplerGl
      id="foo"
      width={width}
      height={height}/>
);
```

##### Component Props

##### `id` (String, required)

- Default: `map`

The id of this KeplerGl instance. `id` is required if you have multiple
KeplerGl instances in your app. It defines the prop name of this KeplerGl state that
stored in the KeplerGl reducer. For example. the state of the KeplerGl component with id `foo` is
stored in `state.keplerGl.foo`

##### `getState` (Function, optional)

- Default: `state => state.keplerGl`

The path to the root keplerGl state in your reducer.

##### `width` (Number, optional)

- Default: `800`

Width of the KeplerGl UI.

##### `height` (Number, optional)

- Default: `800`

Height of the KeplerGl UI.

#### 3. Dispatch custom actions to `keplerGl` reducer.

One greatest advantages of using reducer to handle keplerGl state with vs. react component state is the flexibility
to customize it's behavior. If you only have one `KeplerGl` instance in your app or never intent to dispatch actions to KeplerGl from outside the component itself,
you don’t need to worry about forwarding dispatch and can move on to the next section. But life is full of customizations, and we want to make yours as enjoyable as possible.

There are multiple ways to dispatch actions to a specific `KeplerGl` instance.
- In the root reducer

```
import {keplerGlReducer} from '@uber/kepler.gl';

// Root Reducer
const reducers = combineReducers({
 keplerGl: keplerGlReducer,

 app: appReducer
});

const composedReducer = (state, action) => {
 switch (action.type) {
   case 'QUERY_SUCCESS':
     return {
       ...state,
       keplerGl: {
         ...state.keplerGl,
         // 'map' is the id of the keplerGl instance
         map: visStateUpdaters.updateVisDataUpdater(state.keplerGl.map, {datasets: action.payload})
       }
     };
 }
 return reducers(state, action);
};

export default composeddReducer;
```

- Using redux `connect`

You can add dispatch function to your component that dispatch action to a specific `keplerGl` component,
using connect.


```
import KeplerGl, {// component
  toggleFullScreen, // action
  forwardTo // forward dispatcher
} from '@uber/kepler.gl';

import {connect} from 'react-redux';

Const MapContainer = props => (
  <div>
    <button onClick=() => props.keplerGlDispatch(toggleFullScreen())/>
    <KeplerGl
      id="foo"
    />
  </div>
)

const mapStateToProps = state => state
const dispatchToProps = (dispatch, props) => ({
 dispatch,
 keplerGlDispatch: forwardTo(‘foo’, dispatch)
});

export default  connect(
 mapStateToProps,
 dispatchToProps
)(MapContainer);
```

- Wrap action payload

You can also simply wrap an action into a forward action with the `wrapTo` helper

```
import KeplerGl, {// component
  toggleFullScreen, // action
  wrapTo // forward action wrapper
} from 'mapbuilder';

const wrapToMap = wrapTo('map');
const MapContainer = ({dispatch}) => (
  <div>
    <button onClick=() => dispatch(wrapToMap(toggleFullScreen())/>
    <Mapbuilder
      id="map"
    />
  </div>
);

```

