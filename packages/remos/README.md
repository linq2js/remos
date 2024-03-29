# remos

(RE)act (M)odel (O)riented (S)tore

## Installation

**NPM**

```bash
npm i remos --save
```

**YARN**

```bash
yarn add remos
```

## Features

1. [x] Simple setup
2. [x] No provider required
3. [x] No boilerplate
4. [x] Model inheritance supported
5. [x] Nested model supported
6. [x] Model injection supported
7. [x] Fully Typescript supoorted
8. [x] Immer supported
9. [x] Memoized data supported
10. [x] Model family supported
11. [ ] Redux Devtools supported

## Counter App (5 lines of code) [CodeSandbox](https://codesandbox.io/s/solitary-cherry-5th8g7)

```js
import { useModel } from "remos";
const App = () => {
  const model = useModel({ count: 1 });
  return <h1 onClick={() => model.count++}>{model.count}</h1>;
};
```

## Core Concepts

Imagine the model is where to describe a state and actions of the application. A simple model looks like this:

```js
import { create } from "remos";

const counterModel = create({
  // define count prop and set initial value as 0
  count: 0,
  // define model method
  increase() {
    // update model prop, the 'this' keyword means the current model
    this.count++;
  },
});
// even we can update the model prop outside the model's methods
// but describing method to escapsule the model logics and those logics can re-use easily
counterModel.count++;
```

There are 2 kinds of model, global and local models. A global works like single Redux store, that means we can have multiple stores in an app.
The local model works like useState() hook, you can use it to store local states and when you change its props, the component will re-render

```js
import { create, useModel } from "remos";

// the global is created by calling create function with specified props
const globalModel = create({});

const GlobalModelConsumer = () => {
  // consume globalModel, the component will re-render when global model props are changed
  useModel(globalModel);
  // render global model prop
  return <h1>{globalModel.prop}</h1>;
};

const LocalModelConsumer = () => {
  // create and consume local model
  // local model is created once and get from cache for next re-rendering time
  const localModel = useModel({ prop1: 1, prop2: 2 });
  // if you local model has complex logic and too much properties, you can put those things into the function that returns a model props, that function is called once
  const otherLocalModel = useModel(() => ({
    prop1: 1,
    prop2: 2,
    method1() {},
    method2() {},
  }));
  return <h1>{localModel.prop1}</h1>;
};
```

## Magic methods

You can define magic methods to handle some special events of the model

```js
const model = create({
  firstName: "",
  lastName: "",
  fullName: "",
  // this method will be called when the model has first access (get, set, listen)
  _onInit() {},
  // this method will be called whenever the model's props are changed
  _onChange() {},
  // this method will be called when firstName prop changed
  // that means you can define onXXXChange() to handle changing event of other props
  _onFirstNameChange() {},
  // this method will be called to validate the firstName prop
  _valFirstName() {
    // returning true to mark the firstName prop is VALID
    // returning false to mark the firstName prop is INALID
    // throwing an error to mark the firstName prop is INVALID
    // calling $invalid('firstName', true/false/Error) to set the invalid status of the firstName prop
  },
  // this method will be called after onChange() call,
  // you can put the validation logic for whole model here
  _valAll() {},
  // this is custom getter for fullName
  _getFullName() {
    return `${this.firstName} ${this.lastName}`;
    // if you have complex computation, you can use $memo to cache the result
    // return this.$memo(
    //  () => // do complex computation,
    //  [this.firstName, this.lastName] // this is dependency list, the memo does re-evaluation when the dep values are changed only
    // );
  },
  // if you dont define custom setter for specified prop,
  // that prop become readsonly
  // _setFullName(value: string) {}
});
```

## Binding with React

Using useModel to bind the model to the host component, when the model changes, the host component re-renders

```js
import { useModel } from "remos";
import myModel from "path/to/model";

const App = () => {
  useModel(myModel);
  return <div><{myModel.myProp}/div>;
};
```

## Recipes

### Creating simple model

```js
import { create } from "remos";
// create a model with specified props
// the created model has binded properties
// when binded property changed, the change listeners will be called
const counter = create({ count: 1 });
// register change listener
counter.$listen(() => console.log("changed"));
counter.count++; // update count prop
```

### Connecting model to React component

```js
import { create, useModel } from "remos";

const counter = create({ count: 1 });

const App = () => {
  // bind the model to the component
  // when model is changed, the component re-renders
  useModel(counter);
  return <h1>{counter.count}</h1>;
};
```

### Selecting single model prop

It detects changes with strict-equality (old === new) by default.

```js
import { create, useModel } from "remos";
const counter = create({ count: 1, other: 2 });
const App = () => {
  // select only count prop value
  // that means the component will re-render when count prop change only
  // nothing happens if other prop is changed
  const count = useModel(counter, (props) => props.count);
  return <h1>{count}</h1>;
};
```

### Selecing multiple model props

For more control over re-rendering, you may provide an alternative equality function on the thrid argument.

```js
import { create, useModel } from "remos";
const user = create({ firstName: "", lastName: "", age: 50 });
const App = () => {
  const result = useModel(
    user,
    (props) => (
      {
        firstName: props.firstName,
        lastName: props.lastName,
      },
      // use shallow compare or pass custom equality function here
      "shallow"
    )
  );

  return (
    <h1>
      {result.firstName} {result.lastName}
    </h1>
  );
};
```

### Handling multiple model updates

```js
const model1 = create({});
const model2 = create({});
const model3 = create({});

const App = () => {
  // the component will be re-rendered whenever model1, model2, model3 are updated
  useModel([model1, model2, model3]);
  return <></>;
};
```

### Selecting props from multiple models

```js
const author = create({ name: "Bill" });
const article = create({ title: "React basis" });
const App = () => {
  const result = useModel(
    { author, article },
    (props) =>
      `The article ${props.article.title} is written by ${props.author.name}`
  );
  const { authorName, articleTitle } = useModel(
    { author, article },
    (props) => ({
      authorName: props.author.name,
      articleTitle: props.article.title,
    }),
    "shallow" // using shallow compare if the selector returns complex object
  );
  return <div>{result}</div>;
};
```

## Demos

1. [Counter App](https://codesandbox.io/s/solitary-cherry-5th8g7)
2. [Todo App](https://codesandbox.io/s/remos-todo-qlqmne)
3. [Using React Suspense](https://codesandbox.io/s/remos-suspense-nxj8cc)
4. [Family Model](https://codesandbox.io/s/remos-family-b0yfnu)
5. [Local Model](https://codesandbox.io/s/local-model-fnso1b)
6. [Hacker News](https://codesandbox.io/s/remos-hacker-news-3m6t29)
7. [Search Box](https://codesandbox.io/s/remos-search-box-e8lz3p)
8. [Performance Test](https://codesandbox.io/s/remos-performance-9plx2i)

## Addons

1. [remos-immer](https://www.npmjs.com/package/remos-immer)

## API References

https://linq2js.github.io/remos/
