---
sidebar_position: 2
---

# MobX best practices

- [Documentation](https://mobx.js.org/)
- [Recommended course](https://egghead.io/courses/manage-complex-state-in-react-apps-with-mobx)

In MMM and the one-metadata-frontend we’re using MobX version 5. Version 6 is currently not compatible with our TS and Babel settings, mainly because of incompatibility of decorator proposals.

<aside>
❗ Following guideline was created for class components (when hooks didn’t exist yet). In Function components, which is our best practice now, prefer using react state over MobX state

</aside>

## MobX in React

We are using MobX in our application, so we can use it in React components as well. ~~It is considered best practice to stick to observables over React state.~~ Consider following simple React component:

```jsx
class MyComponent extends React.Component<{}, { counter: number }> {
  this.state = {
    counter: 1
  }

  handleClick = () => {
    this.setState(({ counter }) => ({ counter: counter + 1 }))
  }

  render() {
    const { counter } =  this.state
    return <div onClick={this.handleClick}>{counter}</div>
  }
}
```

We can do the same using MobX:

```jsx
import { action, observable } from "mobx";
import { observer } from "mobx-react";

@observer
class MyComponent extends React.Component {
  @observable
  counter = 1;

  @action
  handleClick = () => {
    this.counter++;
  };

  render() {
    return <div onClick={this.handleClick}>{this.counter}</div>;
  }
}
```

It is bit more readable, but not a big deal for small examples. The biggest advantage comes later when component grows -> we can easily extract the logic to separate model:

```jsx
// MyCounter.ts
import { action, observable } from "mobx";

class MyCounter {
  @observable
  value = 1;

  @action
  increment = () => {
    this.value++;
  };
}
```

```jsx
// MyComponent.tsx
import { observer } from 'mobx-react'

@observer
class MyComponent extends React.Component {
  private counter = new MyCounter()

  render() {
    return <div onClick={this.counter.increment}>{this.counter.value}</div>
  }
}
```

This allows us to:

- Separate logic from the view
- Make it easily testable
- Reuse the same logic across multiple components (even without HoCs, etc.)

### Reacting to props change

When using observables inside React component marked as `@observer`, props are automatically marked as @observable. You can use that for your advantage in calculated props.

```jsx
interface MyComponentProps {
  name: string;
}

@observer
class MyComponent extends React.Component<MyComponentProps> {
  get greetings() {
    const { name } = this.props;
    return `Hello, ${name}`;
  }

  render() {
    return <div>{this.greetings}</div>;
  }
}
```

### Which component should be @observer

Any component which is using any observable inside it’s `render` method should be decorated as `@observer`.

`mobx-react` encourages to use it even without it. It has not been proven that it has any real performance impact. It can even help it in a lot of cases because of the default _pure_ component characteristics + clever `shouldComponentUpdate` implementation.

### Do not dereference of MobX props

Be aware when dereferencing observable variables. It is not a good idea to do it early (outside of listener scope, ie. the `render` method). This is a common mistake occurring usually in React components.

Example:

```jsx
@observer
class MyComponent {
  private someModel = new SomeModelWithObservables()
  private value: any

  componentDidMount() {
    // !!! Dereferencing
    this.value = someModel.value
  }

  render() {
    return this.value
  }
}
```

The component won’t be rerendered on value change. You can use getters or `@computed` values instead:

```jsx
@observer
class MyComponent {
  private someModel = new SomeModelWithObservables()

  get value() {
    return this.someModel.value
  }

  render() {
    return this.value
  }
}
```

## FAQ

### Do not mutate observable outside of model

If you are creating MobX model class, make sure you always mutate all its observables within the class action methods. Making class self-contained makes debugging much more easier and code more predictable.

```jsx
class Counter {
  @observable
  value: number = 0;

  @action
  tick() {
    this.value++;
  }
}

class MyReactComponent extends React.Component {
  counter = new Counter();

  handleClick = () => {
    this.counter.value++; // bad
    this.counter.tick(); // good
  };

  render() {
    return <button onClick={this.handleClick}>Click me!</button>;
  }
}
```

### Do not mutate outside of @action

Mutating observable property is theoretically possible even outside of method marked as an action but it is strongly discouraged. Putting all your code into `@action` decorated method makes your code both more predictable and more performant. Marking method as an action allows MobX to efficiently batch multiple actions into bulk transactions and it also helps to properly display them in MobX dev tools.

### Asynchronous actions

As mentioned above, every state should be mutated inside an action. It can be bit tricky for asynchronous operations. Consider following example:

```jsx
class MyModel {
  @observable
  data: any;

  @action
  fetchData() {
    this.client.fetch().then(
      // !!! This is executed out of scope of an `fetchData` method
      (data) => {
        this.data = data;
      }
    );
  }
}
```

In order to execute the callback within an action as well, we would have to extract it as an another action method or create higher order function of it:

```jsx
class MyModel {
  @observable
  data: any;

  fetchData() {
    this.client.fetch().then(
      // Wrap callback into action function
      action((data) => {
        this.data = data;
      })
    );
  }
}
```

That works great, but what if we want to use `async` / `await`? There is a problem with that because we have to transpile down the `async` function for older browsers. Babel / TS usually does this by creating generators from it, but transpiler doesn’t know a thing about MobX. It will not wrap each generator step in an `action`. We will have to help it a bit with special annotation.

```jsx
class MyModel {
  @observable
  data: any;

  // Magic decorator which hints the transpiler to translate action correctly
  // Basically use it in place of @action for async functions
  @transformToMobxFlow
  async fetchData() {
    this.data = await this.client.fetch();
  }
}
```

**Links:**

- [Related MobX documentation](https://mobx.js.org/best/actions.html)
- [Article about promises and generators](https://hackernoon.com/async-await-generators-promises-51f1a6ceede2)

### When should I make property @observable

Whenever you want something to react to it’s change ;)

### When should I make property @computed

This bit harder than in case of `@observable`. In case of:

```jsx
class MyModel {
  @observable
  a = 1;

  get twice() {
    return this.a * 2;
  }

  @computed
  get threeTimes() {
    return this.a * 3;
  }
}
```

both `twice` and `threeTimes` will work correctly inside of `@observer`. However there is a difference:

- `twice` will be evaluated every time you ask for it (ie. on every render)
- `threeTimes` will be evaluated only if the value of `a` has actually changed (or was not previously calculated)

You can easily use this for memoization of heavily computed values.

Rule of a thumb is, that if you are not sure, you should mark the getter as `@computed`.

### @observer vs PureComponent

One thing which is easy to miss is that `@observer` decorator on React components implements its own `shouldComponentUpdate` method. It by default mimics behavior of `PureComponent` + it tracks changes on observables.

Because of that you don’t have to inherit from `React.PureComponent` when making a component `@observer`. In fact MobX will complain in that case with a warning.

### No side-effects in listeners

If you are going to cause side-effects within `@computed` you are looking for a bad time. Use [autorun](https://mobx.js.org/refguide/autorun.html) or [reaction](https://mobx.js.org/refguide/reaction.html) for that and separate the side-effect from state change.

### Using `@computed` over `autorun` and `reaction`

`autorun` can be powerful tool, but it can make your life miserable because it is harder to debug. Also it can cause unexpected memory leaks if used improperly (see [Disposing `autorun`](about:blank#disposing-autorun-and-reaction)).

In most cases you should be able to implement anything with combination of `@observable` and `@computed` decorators. If you find yourself in need of using `autorun` take some time and try to think if there isn’t other way of modeling the situation.

Here is common anti-pattern:

```jsx
class MyModel {
  @observable
  foo: number

  @observable
  bar!: number

  constructor() {
    // ! Do not do this
    autorun(() => {
      this.bar = foo * 2
    })
  }
}
```

You can easily write it using computed observables

```jsx
class MyModel {
  @observable
  foo: number;

  @computed
  get bar() {
    return this.foo * 2;
  }
}
```

The usual exception to the rule are side-effects.

### Disposing `autorun` (and `reaction`)

In case you really need to use `autorun` (see above [Using @computed over `autorun`](#using-computed-over-autorun-and-reaction)) you have to clean up after it.

Common mistake is to just create it within `componentDidMount` method like:

```jsx
import { autorun } from "mobx";
import { observer } from "mobx-react";

@observer
class MyComponent extends React.Component {
  componentDidMount() {
    // !!! This will cause memory leaks because the listener is not disposed properly
    autorun(() => {
      // Do whatever
    });
  }

  // ...
}
```

Instead dispose it as well:

```jsx
@observer
class MyComponent extends React.Component {
  private freeListeners!: () => void

  componentDidMount() {
    this.freeListeners = autorun(() => {
      // Do whatever
    })
  }

  componentWillUnmount() {
    this.freeListeners()
  }

  // ...
}
```

**Relevant links:**

- https://mobx.js.org/refguide/autorun.html

### Reacting to props

Currently there is an issue when reacting to props in your @observer component. Any prop change will trigger prop recalculation (`autorun`, `computed`, …). Related read:

- https://github.com/mobxjs/mobx-react/issues/404
- https://github.com/mobxjs/mobx-react/issues/380
