# [A Complete Guide to useEffect](https://overreacted.io/a-complete-guide-to-useeffect/)

## Each Render Has Its Own Props and State
Given the component

```jsx
function Counter() {
  const [count, setCount] = useState(0);
 
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

the `count` variable is just a number, it does not do any special data binding. Every time we call `setCount` react will re-render our component with an updated `count` variable.

>**Whenever we update the state, React calls our component. Each render result “sees” its own `counter` state value which is a _constant_ inside our function.**

## Each Render has its own event handlers
Updating the function to 
```jsx
function Counter() {
  const [count, setCount] = useState(0);
 
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
 
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  );
}
```

if we
- increment the counter to 3
- click "Show alert"
- increment the counter to 5

By the time the three seconds have passed and the alert is shown, it will display **3**. This is because the every render of this component has it's own version of `handleAlertClick`, each of which "remembers" it's own version of `count` (3 in this case).

>**Inside any particular render, props and state forever stay the same.**

## Each Render has its own effects
The same logic applies to `useEffect`

```jsx
function Counter() {
  const [count, setCount] = useState(0);
 
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
 
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

**Conceptually, we can imagine that effects are part of the render result**
(They're not actually, but its useful for the mental model we're building)

## Synchronization, not lifecycle
>You should think of effects in a similar way. **`useEffect` lets you _synchronize_ things outside of the React tree according to our props and state.**

## Teaching React to diff our Effects
React will only update DOM nodes that are affected by a change to improve performance. However, React cannot guess what a function does without calling it:

```jsx
function Greeting({ name }) {
  const [counter, setCounter] = useState(0);
 
  useEffect(() => {
    document.title = 'Hello, ' + name;
  });
 
  return (
    <h1 className="Greeting">
      Hello, {name}
      <button onClick={() => setCounter(count + 1)}>
        Increment
      </button>
    </h1>
  );
}
```

I the above example, we do not want React to re-run the effect on a state change, because it would not be necessary. However, React cannot possibly know that without calling the function.

This is where the **dependency array** becomes useful:

```jsx
  useEffect(() => {
    document.title = 'Hello, ' + name;
  }, [name]); // Dependency array
```

> **It’s like if we told React: “Hey, I know you can’t see _inside_ this function, but I promise it only uses `name` and nothing else from the render scope.”**

Now, React will only re-run the effect if one of the values in the dependency array is different between renders.

**Don't lie to react about dependencies**

## What happens when dependencies lie
Assuming this example:

```jsx
function Counter() {
  const [count, setCount] = useState(0);
 
  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
 
  return <h1>{count}</h1>;
}
```

Our goal is to update the `count` by one every second. They idea is to create an interval once that increases the counter and destroy it once.

However, this will only increment `count` once. The reason is we lied about our dependencies, so the effect is never re-run and `count` will always be `0` in the interval callback (as this is the value of count when the effect ran, see [this example](https://codesandbox.io/p/sandbox/aged-thunder-f7wlnr?workspaceId=3843d8e9-fd08-40c0-b260-66f8be8e1691)).

## Two ways to be honest about dependencies
1. Include all dependencies in the array
   In our case, that would fix the issue but also cause `setInterval` to be set and cleared on every render, which is undesirable
2. Change the effect code so that it doesn't need a value that changes more often than we want

## Making effects self sufficient
If we want to get rid of a dependency, we should ask ourselves: "What are we using this dependency for?".
In this example, we use count to be able to call `setCount`. However, we don't have to do it this way, we can use the functional update from of `setState`:

```jsx
  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
```

This allows us to remove the dependency from our effect.

With this, we're encoding the intent rather than the result: Instead of saying "set the state to number X", we say "increment the current state (whatever it is) by one

## Decoupling updates from Actions
For this part, the example will be modified to also take a configurable step into account when increasing the count:

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);
 
  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + step);
    }, 1000);
    return () => clearInterval(id);
  }, [step]);
 
  return (
    <>
      <h1>{count}</h1>
      <input value={step} onChange={e => setStep(Number(e.target.value))} />
    </>
  );
}
```

With `step` in the effects dependencies, this means the interval will be reset and started new every time the value of `step` changes. That's fine, but lets assume we didn't want that to happen.

>**When setting a state variable depends on the current value of another state variable, you might want to try replacing them both with `useReducer`.**

React guarantees `dispatch` exposed by `useReducer` to be constant throughout the component lifetime, so we can refactor our code like this

```jsx

const [state, dispatch] = useReducer(reducer, initialState);
const { count, step } = state;
 
useEffect(() => {
  const id = setInterval(() => {
    dispatch({ type: 'tick' }); // Instead of setCount(c => c + step);
  }, 1000);
  return () => clearInterval(id);
}, [dispatch]);
```

(We could leave out the `dispatch` from the dependency array since it is guaranteed to be constant, but it also won't hurt to specify it)

If this component were dependent on props instead, we could still leverage `useReducer` to prevent the props from becoming a dependency of our effect by moving the reducer function _inside_ our component: 

```jsx
function Counter({ step }) {
  const [count, dispatch] = useReducer(reducer, 0);
 
  function reducer(state, action) {
    if (action.type === 'tick') {
      return state + step;
    } else {
      throw new Error();
    }
  }
 
  useEffect(() => {
    const id = setInterval(() => {
      dispatch({ type: 'tick' });
    }, 1000);
    return () => clearInterval(id);
  }, [dispatch]);
 
  return <h1>{count}</h1>;
}
```

## Moving functions inside effect
Often times, its tempting to omit a function from the dependency array to make sure it's only called once. However, that means we're lying about dependencies again.

The simplest solution is to just move function that are _only used by the effect_ inside the effect. This way, they're no longer a dependency of the effect and we're not lying about our effects dependencies.

Sometimes, functions cannot be moved into the effect for one reason or another. Other options are:
- Move the function outside of the component
- use `useCallback` to make sure the function only changes when necessary (basically tackling the problem "from the other side")
