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


```js
const DoneFrame = (props) => {
	return (
  	<div>
    <br />
    	<h2>{props.doneStatus}</h2>
    </div>
  )
}

const Numbers = (props) => {
	// use lodash to make array of nums from 1-9
	const arrayOfNumbers = _.range(1,10);

  const numberClassName = (number) => {
  	if (props.selectedNumbers.indexOf(number) >= 0) {
    return 'selected';
    }
    if (props.usedNumbers.indexOf(number) >= 0) {
    return 'used';
    }
  }

	return (
  	<div className="card text-center">
    	{arrayOfNumbers.map((number, i) => <span key={i} onClick={() => props.selectNumber(number)} className={numberClassName(number)}>{number}</span>
      )}
    </div>
  );
}

const Answer = (props) => {
	return (
  	<div className="col-5">
    	{props.selectedNumbers.map((num,i) => <span onClick={()=> props.removeSelectedNumber(num)} key={i}>{num}</span>)}
    </div>
  );
}

const Button = (props) => {
	let button;

  switch(props.answerIsCorrect) {
  	case true :
    	button = <button onClick={props.acceptAnswer} className="btn btn-success"><i className="fa fa-check"></i></button>
    	break;
    case false :
    button = <button className="btn btn-danger"><i className="fa fa-times" /></button>
    	break;
    default :
     button = <button onClick={props.checkAnswer} disabled={props.selectedNumbers.length === 0} className="btn">=</button>
    	break;
  }
	return (
  	<div>
    	{button}
    </div>
  );
}

const Stars = (props) => {
	return (
  	<div className="col-5">
    	{_.range(props.numberOfStars).map(i => <i key={i} className="fa fa-star"></i>)}
    </div>
  );
}

const Redraw = (props) => {
	return (
  	<div>
    	<button disabled={props.numRedraws === 0} className="btn btn-warning" onClick={props.redrawStars}>
    	  <i className="fa fa-recycle"></i><br />
        {props.numRedraws}
        </button>
    </div>
  )
}

class Game extends React.Component {
	static randomNumber = () => Math.floor(Math.random() * 9) + 1;

	state = {
  	selectedNumbers : [],
    numberOfStars : Game.randomNumber(),
    answerIsCorrect : null,
    usedNumbers : [],
    numRedraws : 5,
    doneStatus : null
  }

  checkAnswer = () => {
  	this.setState((prevState) => {
    	return {
      	answerIsCorrect : prevState.numberOfStars === prevState.selectedNumbers.reduce((acc,num) => acc + num, 0)
      }
    });
  }

  updateDoneStatus = () => {

  }

  acceptAnswer = () => {
  	this.setState((prevState) => {
    	return {
      	usedNumbers : prevState.usedNumbers.concat(prevState.selectedNumbers),
        selectedNumbers : [],
        answerIsCorrect : null,
        numberOfStars : Game.randomNumber()
      }
    })
  }

  selectNumber = (clickedNumber) => {
  	if (this.state.selectedNumbers.indexOf(clickedNumber) >= 0) {
    return;
    }
  	this.setState((prevState) => {
    	return {
      	answerIsCorrect : null,
      	selectedNumbers : prevState.selectedNumbers.concat(clickedNumber)
      };
    });
  }

  removeSelectedNumber = (clickedNumber) => {
  this.setState((prevState) => {
  	return {
    	answerIsCorrect : null,
    	selectedNumbers : prevState.selectedNumbers.filter((num) => num !== clickedNumber)
    }
  })
  };

  redrawStars = () => {
  	this.setState(prevState => {
    	return {
      	numberOfStars : Game.randomNumber(),
        numRedraws : prevState.numRedraws - 1,
        answerIsCorrect : null,
        selectedNumbers : []
      }
    })
  }

  possibleSolutions = (state) => {
  	const possibleNumbers = _.range(1,10).filter(number => state.usedNumbers.indexOf(number) < 0)
  }

  updateDoneStatus = () => {
  	this.setState(prevState => {
    	if (prevState.usedNumbers.length === 0) {
      return {
      	doneStatus: `You've Done it! Nice!`
      }
      }
      if (prevState.numRedraws === 0 && !(this.possibleSolutions(prevState)){
      	return {
        	doneStatus: 'Not this time! Game Over.'
        }
}
    })
  }

	render() {
  	return (
    	<div>
      	<h3>PlayNine</h3>
        <hr />
        <div className="row">
        	<Stars numberOfStars={this.state.numberOfStars} />
          <div className="col-2">
          <Button selectedNumbers={this.state.selectedNumbers}
          				checkAnswer={this.checkAnswer}
                  answerIsCorrect={this.state.answerIsCorrect}
                  acceptAnswer={this.acceptAnswer} />
          <br />
          <Redraw numRedraws={this.state.numRedraws}
          				redrawStars={this.redrawStars} />
          <br />
          </div>
          <Answer selectedNumbers={this.state.selectedNumbers} removeSelectedNumber={this.removeSelectedNumber} />
      	</div>
  			{this.state.doneStatus ?
        <DoneFrame doneStatus={this.state.doneStatus} /> :
        <Numbers selectedNumbers={this.state.selectedNumbers} selectNumber={this.selectNumber} usedNumbers={this.state.usedNumbers} />
        }
      </div>
    )
  }
}

class App extends React.Component {
	render() {
  	return (
    	<div>
      	<Game />
      </div>
    )
  }
}

ReactDOM.render(<App />, mountNode);
```
