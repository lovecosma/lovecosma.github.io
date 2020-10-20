---
layout: post
title:      "User authentication for React-Redux and Rails API: React-Redux Side"
date:       2020-10-20 20:30:26 +0000
permalink:  user_authentication_for_react-redux_and_rails_api_react-redux_side
---


 In my previous post, I set up a user model in a Rails API. In this post, we will discuss how to complete user authentication in React-Redux for a single page application, and how to persist login through a refresh of the page. 
 
 The dependencies for this in addition to the properly configured Rails API are:

'react'
'redux'
'react-dom
'react-redux'
'redux-thunk'

First, let us create a new react application from the terminal. I like to run this in the same directory as my API, but it's up to you:

```
create-react-app user-auth-client
```

Then add the dependencies:

```
yarn add react-dom redux react-redux redux-thunk
```


Now our React app is has been created. If we run 'yarn start', an instance of a web browser should be opened with the default react homepage on it.  I like to slim down the default react app to look like the following:

```
/src/index.js


import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(
<React.StrictMode>
<App />
</React.StrictMode>,
document.getElementById('root')
);

```


```
/src/App.js

import React from 'react';

function App() {
return (
<div>
<h1>Login</h1>
</div>
);
}

export default App;

```


Now we must import our dependencies and set up our index to provide the store to the rest of our app. The store will be the centralized storage for all of our components. Provider will make the store available to all other child components of the app.


```

/src/App.js

import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from './reducers/rootReducer';
import App from './App'


const store = createStore(rootReducer, applyMiddleware(thunk))

ReactDOM.render(
<React.StrictMode>
<Provider store={store} >
<App/>
</Provider>
</React.StrictMode>,
document.getElementById('root')
);
```

You may notice that the function createStore requires a reducer as a first argument. As the second, we are passing in our middleware 'thunk' which will allow us to make fetch requests asynchronously when we need to. Let us create the rootReducer from the specified path in the import. Make a new folder in src called reducers and make a new file inside called 'rootReducer.js'. 



```
/src/reducers/rooReducer.js


import { combineReducers } from 'redux'
import usersReducer from './usersReducer'



const rootReducer = combineReducers({
usersReducer
})

export default rootReducer
```

Here we are making a rootReducer that will combine all other future reducers using combineReducers from redux. We've imported the reducer we will need immediately 'usersReducer'. This reducer contains the store and  handle all tasks related to the user. Next, let's actually make the usersReducer.


```
/src/reducers/usersReducer

function usersReducer(state = { isLoggedIn: false, user: {},  requesting: false } , action) {
switch (action.type) {

case 'START_ADDING_USER_REQUEST':
return {
...state,
requesting: true
}

case 'LOGIN':
return {
isLoggedIn: true,
user: {...action.user},
requesting: false
}

case 'LOGOUT':
return {
isLoggedIn: false,
user: {},
requesting: false
} 
default:
return state;
}
};


export default usersReducer
```

The users reducer will contain info about whether or not a user is loggedIn, which user, and whether the info has been retrieved from the server yet. These are the basic actions that will need to create a session for a user. Next, let's set up the actions that will trigger changes in this reducer. Add a new directory to src called 'actions' and inside create a new file called 'userLogin.js'

```
/src/actions/userLogin.js


const userLogin = user => {
return dispatch => {
dispatch({ type: 'START_ADDING_USER_REQUEST' })
const formData = {
user
}
const configObj = {
method: "POST",
headers: {
"Accept": "application/json",
"Content-Type": "application/json"
},
body: JSON.stringify(formData)
}
fetch('http://localhost:3001/login', configObj)
.then(resp => resp.json())
.then(user => {
console.log(user)
dispatch({type: 'LOGIN', user})
})
.catch(error => console.log('api errors:', error))
};
}

export default userLogin
```

Here, we are making a function 'userLogin' that takes in user information and returns multiple dispatches to 'usersReducer'. The first dispatch lets the reducer know we are loading info. Then, a fetch request is made to the login path of our Rails API which triggers the 'create#sessions' action. Remember, the controller is expecting a user object with keys for email and password. 

When a response is received, it will trigger the 'LOGIN' action of the userReducer sending in the user whose session was just created in the back end.  However, before we can trigger this userLogin action we must call userLogin from a component. Create a folder called components in src and add to it LoginForm.js

```
/src/components/LoginForm.js

import React, { Component } from 'react'
import  userLogin from '../actions/userLogin'
import { connect } from 'react-redux'

export class LoginForm extends Component {

state = {
email: "",
password: ""
}

handleChange = event => {
this.setState({
[event.target.name]: event.target.value
})
}

handleSubmit = event => {
event.preventDefault()
this.setState({
email: "",
password: ""
})
this.props.userLogin(this.state)
}

render() {
return (
<div>

<div>
<h2>Login</h2>
<form onSubmit={this.handleSubmit}>
<div >
<input onChange={this.handleChange} type="text" name="email" placeholder="Email" style={{display: "block"},{width: "750px"}}></input>
</div>
<div>
<input onChange={this.handleChange} type="password" name="password" placeholder={'Password'} style={{display: "block"},{width: "750px"}}></input>
</div>
<button type='submit' name='submit' id='submit'>Login</button>
</form> 
</div>
</div>
)
}
}



export default connect(null, { userLogin })(LoginForm)

```

The LoginForm component creates a minimal login form to be rendered. On change events on the inputs update the state with the values for email and password. On submit, the form calls userLogin and sends it the email and password in the state. 

We're almost ready to send our first request to the API from React-Redux with the login info.  Let's add the login form to App.js

```
/src/App.js

import React from 'react';
import LoginForm from './components/LoginForm'

function App() {
return (
<div className="App">
<LoginForm/>
</div>
);
}

export default App;


```


Now running 'yarn start' should open a browser with a plain login form. This is the moment of truth. When we enter information for the sample user created in the previous post, we should receive a log to the console with the corresponding user object in the browser console.

```
{id: 1, email: "test@test.com", password_digest: "$2a$12$Y0t2nkuFNOYHEQhsIKPK1u49Z0XN9f3sD3MH2idhp6Qd/qUq353n.", created_at: "2020-10-20T01:06:20.158Z", updated_at: "2020-10-20T01:06:20.158Z"}
```

Now, whenever you call on the usersReducer, as long as the data is loaded, you will be able to access info about the user and whether or not they are loggin in. To logout, you just create a second action that posts to the logout route in Rails. However, what if the user refreshes the page. Since this is technically a single page application made with javascript, the information about the user will disappear. To avoid this, we will implement just a few lines of code to check if a user is logged in when the app is rendered.


in userLogin:

```

/src/actions/userLogin

const userLogin = user => {
return dispatch => {
dispatch({ type: 'START_ADDING_USER_REQUEST' })
const formData = {
user
}
const configObj = {
method: "POST",
headers: {
"Accept": "application/json",
"Content-Type": "application/json"
},
body: JSON.stringify(formData)
}
fetch('http://localhost:3001/login', configObj)
.then(resp => resp.json())
.then(user => {
if (user) {
localStorage.setItem('user', JSON.stringify(session.user)) // Store the user locally
dispatch({type:"LOGIN", user}) //Then log them in
} else {
console.log(user.errors)
}
})
.catch(error => console.log('api errors:', error))
};
}

export default userLogin


```



```
...
localStorage.setItem('user', JSON.stringify(session.user)) // Store the user locally
...
```


This line stores the current user's information locally in a key called user. We can read this key when the app loads to determine is a user was logged in or not. Also, we will clear this key when a user actually does log out.


```

/src/App.js

componentDidMount = () => { 
const loggedInUser = JSON.parse(localStorage.getItem("user"));
if(loggedInUser){
this.props.continueSession(loggedInUser)
} else {
// routerProps.history.push('/login')
}
}
```

In my case, I added a componentDidMount function to App.js. When mounted a variable that holds the information that is found in the user key of the local storage. By the way,** the value in this storage needs to be a string**. This is why I convert the object to a string when storing, and parse the JSON from that string when recalling it.  I also created another action that basically continues the session for that user in local memory. Again, when a user actually logs out, the information is wiped locally,

```
/src/actions/userLogout.js


const userLogout = () => {
return dispatch => {
dispatch({ type: 'START_ADDING_USER_REQUEST' })
const configObj = {
method: "POST",
	headers: {
	"Accept": "application/json",
	"Content-Type": "application/json"
	}
}
fetch('http://localhost:3001/logout', configObj)
.then(resp => resp.json())
.then(session => {
if (session.status === 200) {
dispatch({type:"LOGOUT"})
localStorage.removeItem("user")
} else {
console.log(session.errors)
}
)
.catch(error => console.log('api errors:', error))
};
}

export default userLogout
```


This concludes our run-through of user authentication with a Ruby on Rails API and React-Redux. It's a lengthy process, but it's simple when you realize that you're merely making fetch requests for objects like we are used to doing with an API anyway.# Enter your title here

The content of your blog post goes here.
