---
layout: post
---

https://docs.expo.io/versions/v34.0.0/introduction/additional-resources/

https://reactjs.org/tutorial/tutorial.html

```
npx create-react-app tic-tac
yarn start
```

## React Component

https://reactjs.org/docs/react-component.html
`React.Component` receive `props` and returns view `render()`.

* `render()` only required method
* `this.props` and `this.state` is available on component. Treat `this.state` as
  immutable and always use `setState()`
* Use props `on[Event]` for events and `handle[Events]` for even handleres.


Here is example of *controlled compoment* (they do not contain the state, just
receive value and call the callback function)

```
class Square extends React.Component {
  render() {
    return (
      <button
        className="square"
        onClick={() => this.props.onClick() }
      >
        {this.props.value}
      </button>
    );
  }
}
```

Here is example of *function component*

```
function Square(props) {
  return(
    <button
      className="square"
      onClick={props.onClick}
    >
      {props.value}
    </button>
  )
}
```

To collect data from multiple children, or have two child compoments communicate
between each other, you need to declare state in parent component and pass the
state back to children with `props`.

## JSX

https://reactjs.org/docs/jsx-in-depth.htm
JSX transforms `<div/>` to `React.createElement('div', { className: '' })`.
Inside `{}` can be js object.

