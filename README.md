### Component

* reusable
* can contain other components
* take properties
* can have state

### Reactive Updates

* react 'reacts' to state changes and updates the DOM

### Virtual Views In Memory

* write HTML with JavaScript (instead of modify HTML w/ JS)
* Virtual DOM -> version of HTML in memory, react optimizes by using tree reconciliation to change parts of the DOM (not rerender the whole doc)

Function Component vs Class components

* Class components have a private internal state

* Function components take props and those props dictate changes in values within the component

example:

```js
class Button extends React.Component {
	constructor(props) {
  	super(props);
  }

  // use named functions as callbacks to prevent a new one be created every time
  handleClick = () =>  {
  	this.props.onClickFunction(this.props.incrementValue)
  }

	render() {
  	return (
			<button onClick={this.handleClick}>
      	+{this.props.incrementValue}
      </button>
		);
  };
}

const Result = (props) => {
	return (
  	<div>{props.counter}</div>
  )
}

// Use coordinator components to display components as siblings
class App extends React.Component {
	state = {
  	counter : 0
  };

  incrementCounter = (incrementValue) => {
  	this.setState((prevState) => {
    	return {
      	counter : prevState.counter + incrementValue
      }
    })
  }

	render() {
  	return (
    	<div>
        // example of function and property bindings in React
      	<Button onClickFunction={this.incrementCounter} incrementValue={1} />
      	<Button onClickFunction={this.incrementCounter} incrementValue={10} />
      	<Button onClickFunction={this.incrementCounter} incrementValue={100} />
        <Result counter={this.state.counter}/>
      </div>
    )
  }
}

ReactDOM.render(<App />, mountNode);

```

### Working with Data
==> Github API


```js
// Presentation Detail Component
const Card = (props) => {
  return (
    <div>
      <img width="75" src={props.user.avatar_url} />
      <div className="info">
        <div>{props.user.login}</div>
        <div>{props.user.company}</div>
      </div>
    </div>
  )
}

// Presentation List Component
const CardList = (props) => {
  return (
    <div>
      // React (like angular) wants unique identifiers("key") for each component
      {props.users.map(user => <Card key={user.id} user={user} />)}
    </div>
  );
}

// Sibling to list, uses function provided by parent to update list
class Form extends React.Component {
  // set initial state
  state = {
    username : ''
  }

  // call back for form submition
  handleSubmit = (event) => {
    event.preventDefault();
      // make async request to api with user input
      // NOTE: axios uses the Promise API (sweet)
      axios
        .get(`https://api.github.com/users/${this.state.username}`)
        .then((res) => {
          // pass response data to parent via function binding
          this.props.addUser(res.data)
          // reset initial state
          this.state.username = ""
        })
  };

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <input type="text" value={this.state.username}
          onChange={(event)=> this.setState({username : event.target.value})}
          placeholder="Github Username" />
        <button type="submit">Add Card</button>
      </form>
    )
  }
}

// App Component
class App extends React.Component {
  // setting initial state
  state = {
    users : []
  }

  // function to bind to child component
  addUser = (user) => {
    // deal asynchronously with state to avoid race condition
    this.setState((prevState) => {
      return {
        users : prevState.users.concat(user)
      }
    })
  }

  render () {
    return (
      <div>
        // pass the form the function
        <Form addUser={this.addUser} />
        // pass the list the users
        <CardList users={this.state.users} />
      </div>
    )
  }
}

ReactDOM.render(<App />, mountNode);
```
