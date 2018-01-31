![](contextual.jpg)

react-contextual is a tiny (~1KB) [at least when React 16.3.0 drops] store/hoc pattern around [React 16's new context API](https://github.com/acdlite/rfcs/blob/new-version-of-context/text/0000-new-version-of-context.md). 

It provides two things:

1. a minimal redux-like flux store with setState semantics and central actions
2. helping you to deal with context in general, listening to multiple providers without the nesting, etc.

# Why

In the upcoming version React is going to have a new, powerful low-level API for dynamic context distribution. The API is built on render props. While they are very powerful they can make the codebase unwieldy, especially if you work with several providers that will cover consumers in scores of nested blobs. react-contextual can fix that by mapping context values to component props, similar to how Redux operates.

Likewise, context makes flux patterns possible that previously would have meant larger dependencies and boilerplate. Context can carry setState to new heights by allowing it to freely distribute. react-contextual builds a small flux pattern around that premise but lets React do all the work, which perhaps leads to what could well be [the smallest flux-store yet](https://github.com/drcmda/react-contextual/blob/master/src/store.js).

# Installation and usage

    npm install react-contextual

## 1. If you just need a light-weight store ...

Example: https://codesandbox.io/s/ko1nz4j2r

Provide your state, wrap everything that is supposed to access or mutate it within.

```js
import React from 'react'
import ReactDOM from 'react-dom'
import { StoreProvider } from 'react-contextual'
import TestStore from './TestStore.js'

ReactDOM.render(
    <StoreProvider
        initialState={{ name: 'max', age: 99,  }}
        actions={{
            setName: name => ({ name }), // simple merge
            setAge: age => state => ({ age: state.age + 1 }), // functional merge with more access
        }}>
        <TestStore />
    </StoreProvider>,
    document.getElementById('app'),
)
```

Consume anywhere within the provider, as deeply nested as you wish. The semantics are similar to Redux.

```js
import React from 'react'
import { connectStore } from 'react-contextual'

class TestStore extends React.PureComponent {
    render() {
        const { name, age, actions } = this.props
        return (
            <div>
                <button onClick={() => actions.setName('paul')}>{name}</button>
                <button onClick={() => actions.setAge(28)}>{age}</button>
            </div>
        )
    }
}

export default connectStore(
    // Pick your state, map it to the components props, provide actions ...
    ({ state, actions }) => ({ name: state.name, age: state.age, actions })
)(TestStore)
```

### With decorator

Makes it a little more tidy, but use with care, as the spec may still change any time!

```js
@connectStore(({ state, actions }) => ({ name: state.name, age: state.age, actions }))
export default class TestStore extends React.PureComponent {
    render() {
        ...
    }
}
```

## 2. Dealing with raw context providers of any kind

Example: https://codesandbox.io/s/5v7n6k8j5p

You can use contextuals `context` HOC to listen to one or multiple React context providers. Their values will be mapped to regular props, similar to how Redux operates. You provide context as you normally would, look into Reacts [latest RFC](https://github.com/acdlite/rfcs/blob/new-version-of-context/text/0000-new-version-of-context.md) for more details.

Make the consuming component a PureComponent and you get shallowEqual prop-checking for free, in other words, it only renders when the props you have mapped change.

```js
import React from 'react'
import { context } from 'react-contextual'

class Test extends React.PureComponent {
    render() {
        const { theme, count } = this.props
        return (
            <h1 style={{ color: theme === 'light' ? '#000' : '#ddd' }}>
                Theme: {theme} Count: {count}
            </h1>
        )
    }
}

export default context(
    // Pick one or several contexts, then map the values to the components props ...
    [ThemeContext, CounterContext], ([theme, count]) => ({ theme, count })
)(Test)
```

### With decorator

```js
@context([ThemeContext, CounterContext], ([theme, count]) => ({ theme, count }))
export default class Test extends React.PureComponent {
    render() {
        ...
    }
}
```

# API

https://github.com/drcmda/react-contextual/blob/master/API.md
