# MERN STACK Project

update/upgrade Ubuntu

## Installed Node.js
`curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -` - identify Node.js location in Ubuntu repository

`sudo apt-get install -y nodejs`

![Node.js installed](./images/verify_node_installed.png)

`mkdir Todo` - created new directory for Todo project

`cd Todo` navigate to new directory

`npm init` installed package.json

![package.json installed](./images/package_json.png)


## Installed ExpressJS

`npm express` - installed express using npm

`touch index.js` - created a index.js file

`Install the dotenv module` - Installed the dotenv module

`vim index.js` Open the file and paste the following

```
const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```

save `:w` code in vim and exit `:qa`

`node index.js` Start Server

![server started](./images/package_json.png)

Next, open the port 5000 in Ec2 security group

![port 5000 enabled](./images/port_5000.png)

Tested result with `http://<PublicIP-or-PublicDNS>:5000`

![test result](./images/welcome_express.png)

Next created *ROUTES*

There are three actions that our To-Do application needs to be able to do:

- Create a new task
- Display list of all tasks
- Delete a completed task

Each task will be associated with some particular endpoint and will use different standard `HTTP request methods`: POST, GET, DELETE.

For each task, we need to create *routes* that will define various endpoints that the *To-do* app will depend on. So let us create a folder *routes*

`mkdir routes` - created routes directory

`cd routes`

Next, created a file `api.js` with the command below

`touch api.js`

`vim api.js` - open the file and paste the code below

```
const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```

## Models

change back to Todo directory - `cd Todo`

Install mongoose to create schema and a model

`npm install mongoose`

create new *model* directory, change into the directory and create *todo* file

`mkdir models && cd models && touch todo.js`

`vim todo.js` - open the file and paste the code below

```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```
Next update *route* directory to update the `api.js` file to use the model

Open *api.js* with ~vim api.js~, delete the code inside with `:%d` command and paste there code below into it then save and exit `:wq`.

```
const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {

//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});

router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});

router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})

module.exports = router;
```
## MongoDB Database
To implement MERN STACK, a database is required to store my data. I utilized mLab a cloud database service to host my MongoDB Database. I created an account on myLab (www.mongodb.com), signed in and created a database in Mongo DB. 

Next I created *.env* file in my *Todo* directory. Inside the file entered the connection string required to connect to the database using

`DB = mongodb+srv://test_db:Test123@cluster0.ljrzt.mongodb.net/TestDB?retryWrites=true&w=majority'`


Updated index.js file to make use of the .env file, This will allow Node.js to connect to my database.

Updated the index.js file with the code below
```
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
console.log(err);
next();
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```
started *node.js*

![MongoDB connected successfully](./images/mongodb_connected.png)

### Testing Backend Code without Frontend using RESTful API
Used postman to test that the API endpoints are working. Used the POST operation to created a new record in DB

![Postman header configuration](./images/postman_post_header.png)

![Postman Post operation](./images/postman_post_body.png)

![Postman post operation result](./images/postman_post_success.png)

With Postman I was able to perform the following tasks

- Display a list of task - HTTP GET request
- Add a new task to the list – HTTP POST request

## FrontEnd Creation

Haven completed functionality of the backend and API, I then created the user interface and web client (browser) to interact with the API

### Runnung a React App

`npx create-react-app client` 

Installed other dependencies

`npm install concurrently --save-dev`- installed *concurrently* so I can run multiple commands simulteneously.

Installed *nodemon*,which is used to run and monitor the server.

`npm install nodemon --save-dev`

Updated the *package.json* file, replaced the highted part

 

with
```
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
```
### Configure proxy in package.json

Change into client directory from Todo directory
`cd client`.

Next update the *package.json* file and add *"proxy": "http://localhost:5000",* at begining within code block.

"proxy": "http://localhost:5000". This is to access the application directly from the browser by simply calling the server url like http://localhost:5000

Ran command `npm run dev` in the Todo directory, to verify that my app will open and start running on localhost:3000. Finally opened TCP port 3000 on my EC2 instance.

![configured proxy](./images/proxy_configured.png)

Screenshot showing access from my web browser.

![browser webapp access](./images/webapp_access.png)

### Creating your React Components

Two stateful components and one stateless component were setup. While in the Todo/client/src directory, i setup a folder called components

`cd Todo/client/src`

`mkdir components` create a new directory

`cd components`

in the *component directory* I created 3 files

`touch Input.js ListTodo.js Todo.js`

I pasted the under codes in Input.js (`vi Inputs.js`)

```
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {

state = {
action: ""
}

addTodo = () => {
const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }

}

handleChange = (e) => {
this.setState({
action: e.target.value
})
}

render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}

export default Input
```

Installed Axios (Axios is a Promise based HTTP client for the browser and node.js) inside the *Todo/client directory*

In *src/components directory* open *ListTodo.js* file and entered the following

```
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {

return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}

export default ListTodo;
```

Opened *Todo.js* file and entered the code below:

```
import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {

state = {
todos: []
}

componentDidMount(){
this.getTodos();
}

getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}

deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))

}

render() {
let { todos } = this.state;

    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )

}
}

export default Todo;
```
In the src folder, opened the *App.js* file and deleted the exiting code and then entered the code below.
```
import React from 'react';

import Todo from './components/Todo';
import './App.css';

const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}

export default App;
```
Next opened the *App.css* file, delete the exiting code and enter the below code
```
.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}

input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}

input:focus {
outline: none;
}

button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}

button:focus {
outline: none;
}

ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}

li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}

@media only screen and (min-width: 300px) {
.App {
width: 80%;
}

input {
width: 100%
}

button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}

@media only screen and (min-width: 640px) {
.App {
width: 60%;
}

input {
width: 50%;
}

button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}
```
Next opened the *index.css* file, delete the exiting code and enter the below code
```
body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}

code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}
```
Finally, in the $Todo directory, I ran the command below to start Express.js server.

`npm run dev`

![start service](./images/completed_express.png)

Confirmation on browser:

![Browser](./images/browser_test.png)

With my To-do application successfully deployed on a MERN Stack, I was able to perform the following tasks

- Add new task (project 1, Design Doc)
- Delete task

URL: `http://<public address>:3000`



