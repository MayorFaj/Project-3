# Documentation of Project-3 #

# STEP 1 #

## **Backend Configuration** ##

`sudo apt update`

`sudo apt upgrade`

- Retrieve Node.js software location from Ubuntu repositories

`curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -`

![Node.JS Installation](./Images/Retrieveing%20Node.Js%20Location.png)

`sudo apt-get install -y nodejs`

`node -v`

![Node Verification](./Images/Node%20Instal%20Verification.png)

`npm -v`

![node package manager verification](./Images/npm%20install%20verification.png)

`mkdir Todo`

`ls`

- Run **ls -lih** to see more useful information about files or directory

`ls -lih`

![Todo directory](./Images/Todo%20directory%20created.png)

`cd Todo`

- use **npm init** to initialise project to create a new file named **package.json** 

`npm init`

![Node package manager initialized](./Images/npm%20initialize%20status.png)

`ls`

![Package.json File](./Images/Package.Json%20File%20initialized.png)



## **Install ExpressJS** ##

`npm install express`

![Express install](./Images/Express%20install.png)

- create a file index.js

`touch index.js`

- Run **ls** to confirm that the index.js file is successfully created
`ls`

- Install the dotenv module

`npm install dotenv`

![dotenv status](./Images/dotenv%20install.png)
 
 - open the vim index.js file

`vim index.js`

> paste

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

- Open the terminal in the same directory as the index.js file and type

`node index.js`

![Sever running prompt](./Images/Port5000%20running.png)

- open port 5000 in EC2 Security Groups 

- Open the browser and access the server's public IP followed by port 5000 (http://<PublicIP-or-PublicDNS>:5000)

![Express](./Images/Welcome%20Express%20promt.png)

- There are three actions that our To-Do application needs to be able to do:
Create a new task,Display list of all tasks, Delete a completed task. Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE.
- For each task, we need to create routes that will define various endpoints that the To-do app will depend on. Create a folder routes.


`mkdir routes`

`cd routes`

- create a file api.js

`touch api.js`


`vim api.js`

- paste

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



## **Models** ##
- change directory to Todo

`cd Todo`

`npm install mongoose`

![Mongoose install](./Images/Mongoose%20install.png)

`mkdir models && cd models && touch todo.js`

- Open file created 

`vim todo.js`

> Paste below code in the file

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

- there is need to update our routes from the file api.js in routes directory to make use of the new model

- change directory to routes, open api.js with `vim api.js` , delete the code with :%d,  and paste below code into it the save and exit

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

![cat](./Images/CatConfirm.png)

## **MONGODB DATABASE** ##
- there is need for a database where data will be stored, as such we'll make  use of *mLab*. mLab provides MongoDB database as a service solution.
- sign up for a shared cluster free account

- create a file in the Todo directory .env

 `touch .env`

 `vi .env`

 - add conneection string to access database

```
DB = mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority
```

- There is need to update the index.js to reflect the use of .env so that node.js can connect    to the database
-Delete the existing content in the file, and update it with the code below

`vim index.js`

> Paste

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

`node index.js`

![Database connected](./Images/Database.%20Connected.png)

- now we have our backend configured

- Next is to test the backend code using RESTful API

1. Install Postman
2. Create a POST request to the API http://<PublicIP-or-PublicDNS>:5000/api/todos

![POST request](./Images/POST%20reequest.png)

3. Set header key **Content-Type** as **application/json**
4. Create a GET request to your API on http://<PublicIP-or-PublicDNS>:5000/api/todos

![GET request](./Images/GET%20request.png)


# STEP 2 #

## **FRONTEND CONFIGURATION** ##

- In the Todo directory, create a new folder called client

`npx create-react-app client`

![client](./Images/create-react-app1.png)

`npm install concurrently --save-dev`

![concurrently](./Images/Install%20concurrently.png)

`npm install nodemon --save-dev`

![nodemon](./Images/Install%20nodemon.png)

- in the Todo folder, open the **package.json** file, edit and paste the code below

`vi package.json`

```
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
```

- configure proxy in **package.json**

`cd client`

`vi package.json`

- add the value key pair 

``` 
"proxy": "http://localhost:5000" 
```

- cd to Todo

`npm run dev`

![port 3000 up](./Images/npm-run-dev.png)

- Note: open TCP port 3000 on EC2 to access the application from internet

![internet npm run](./Images/client%20in%20browser.png)

- from Todo directory, change directory to client

`cd client`

- move directory to source

`cd src`

`mkdir components`

`cd components`

`touch Input.js ListTodo.js Todo.js`

`vi Input.js`

> paste

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

- cd to client to install axios a promised based HTTP client for the browser and node.js

`cd ../..`

`npm install axios`

`cd src/components`

`vi ListTodo.js`

- paste

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

export default ListTodo
```

`vi Todo.js`

- paste

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

- there is need to adjust the react code, delete the logo, and adjust the **App.js** 

- change directory to src

`vi App.js`

> paste

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

`vi App.css`

> paste

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

`vim index.css`

> paste

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

- change directory to Todo

`cd ../..`

`npm run dev`

![Todo app](./Images/TodoProjet3.png)

