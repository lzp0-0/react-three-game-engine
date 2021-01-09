# react-three-game-engine

[![Version](https://img.shields.io/npm/v/react-three-game-engine?style=flat&colorA=000000&colorB=000000)](https://www.npmjs.com/package/react-three-game-engine)
[![Downloads](https://img.shields.io/npm/dt/react-three-game-engine.svg?style=flat&colorA=000000&colorB=000000)](https://www.npmjs.com/package/react-three-game-engine)

A very early experimental work-in-progress package to help provide game engine functionality for [react-three-fiber](https://github.com/pmndrs/react-three-fiber).

### Key Features
- [planck.js](https://github.com/shakiba/planck.js/) 2d physics integration
- Physics update rate independent of frame rate
- `OnFixedUpdate` functionality
- Additional React app running in web worker, sync'd with physics, for handling game logic etc

### Note
The planck.js integration currently isn't fully fleshed out. I've only been adding in support 
for functionality on a as-needed basis for my own games.

Also if you delve into the source code for this package, it's a bit messy!

## Get Started

### General / Physics

1. Install required packages

`npm install react-three-game-engine`

plus

`npm install three react-three-fiber planck-js`

2. Import and add `<Engine/>` component within your r3f `<Canvas/>` component. 

```jsx
import { Engine } from 'react-three-game-engine'
import { Canvas } from 'react-three-fiber'
```

```jsx
<Canvas>
    <Engine>
        {/* Game components go here */}
    </Engine>
</Canvas>
```

3. Create a planck.js driven physics body

```jsx
import { useBody, BodyType, BodyShape } from 'react-three-game-engine'
import { Vec2 } from 'planck-js'
```

```jsx
const [ref, api] = useBody(() => ({
    type: BodyType.dynamic,
    position: Vec2(0, 0),
    linearDamping: 4,
    fixtures: [{
        shape: BodyShape.circle,
        radius: 0.55,
        fixtureOptions: {
            density: 20,
        }
    }],
}), {})
```

4. Control the body via the returned api

```jsx
api.setLinearVelocity(Vec2(1, 1))
```

5. Utilise `useFixedUpdate` for controlling the body

```jsx
import { useFixedUpdate } from 'react-three-game-engine'
```

```jsx

const velocity = Vec2(0, 0)

// ...

const onFixedUpdate = useCallback(() => {

    // ...

    velocity.set(xVel * 5, yVel * 5)
    api.setLinearVelocity(velocity)

}, [api])

useFixedUpdate(onFixedUpdate)

```

### React Logic App Worker

A key feature provided by react-three-game-engine is the separate React app running 
within a web worker. This helps keep the main thread free to handle rendering etc, 
helps keep performance smooth.

To utilise this functionality you'll need to create your own web worker. You can 
check out my repo [react-three-game-starter](https://github.com/simonghales/react-three-game-starter) 
for an example of how you can do so with `create-react-app` without having to eject.

1.

Create a React component to host your logic React app, export a new component wrapped with 
`withLogicWrapper`

```jsx
import {withLogicWrapper} from "react-three-game-engine";

const App = () => {
    // ... your new logic app goes here
}

export const LgApp = withLogicWrapper(App)
```

1.

Set up your web worker like such 
(note requiring the file was due to jsx issues with my web worker compiler)

```jsx
/* eslint-disable no-restricted-globals */
import {logicWorkerHandler} from "react-three-game-engine";

// because of some weird react/dev/webpack/something quirk
(self).$RefreshReg$ = () => {};
(self).$RefreshSig$ = () => () => {};

logicWorkerHandler(self, require("../path/to/logic/app/component").LgApp)
```

1.

Import your web worker (this is just example code)

```jsx
const [logicWorker] = useState(() => new Worker('../path/to/worker', { type: 'module' }))
```

1.

Pass worker to `<Engine/>`

```jsx
<Engine logicWorker={logicWorker}>
    {/* ... */}
</Engine>
```

You should now have your Logic App running within React within your web worker, 
synchronised with the physics worker as well. 

### Controlling a body through both the main, and logic apps.
