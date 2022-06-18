# PROJECT 3: MERN STACK IMPLEMENTATION
**A SIMPLE TO-DO APPLICATION ON MERN WEB STACK**

- **Task :**
To deploy a simple To-Do application that creates To-Do lists like this:
![mern](https://user-images.githubusercontent.com/107736487/174439588-d184c11d-2c1b-448b-a969-d6aa4038ce73.gif)

## STEP 1 – BACKEND CONFIGURATION
---
- Update ubuntu

  `sudo apt update`

- Upgrade ubuntu

  `sudo apt upgrade`

- Lets get the location of Node.js software from Ubuntu repositories.

  `curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -`  

- Install Node.js on the server

  `sudo apt-get install -y nodejs`

  Note: The command above installs both nodejs and npm. NPM is a package manager for Node like apt for Ubuntu, it is used to install Node modules & packages and to manage dependency conflicts.

- Verify the node installation with the command below

  `node -v`

  ![verify node installation](https://user-images.githubusercontent.com/107736487/174439643-a093f65b-fb20-4ff6-93ba-2499ea773c46.PNG)

- Verify npm installation with the command below

  `npm -v` 

  ![npm version](https://user-images.githubusercontent.com/107736487/174439676-db0cf359-c77c-4bb6-a54a-e1437c4293fe.PNG)

- **Application Code Setup**

1. Create a new directory for your To-Do project:

   `mkdir Todo`

   ![Todo Dir](https://user-images.githubusercontent.com/107736487/174439698-0c63731e-fb03-4086-8adf-9ba7bf3c41e7.PNG)

2. Now i'll change my current directory to the    newly created one:

   `cd Todo`   

3. Next, i'll use the command npm init to  initialise my project, so that a new file named **package.json** will be created. 
   
   This file will normally contain information about my application and the dependencies that it needs to run. I follow the prompts after running the command, accept default values by pressing enter, then accept to write out the package.json file by typing yes.

   `npm init`

   Run the command ls to confirm that i have **package.json** file created.

   ![ls](https://user-images.githubusercontent.com/107736487/174439722-85a03366-5d97-47c8-843e-981efba47bda.PNG)

## INSTALL EXPRESS JS AND CREATE THE ROUTES DIRECTORY. 
---
- Install express using npm:

  `npm install express`

- Create a file **index.js** with the command below

  `touch index.js` 

- Install the dotenv module

  `npm install dotenv`

- Open the index.js file with the command below

  `vim index.js`

  Type the code below into vi editor and save.
  ```
   const express = require('express');
   require('dotenv').config();

   const app = express();

   const port = process.env.PORT || 5000;

   app.use((req, res, next) => {
   res.header("Access-Control-Allow-Origin", "\*");
   res.header("Access-Control-Allow-Headers",    "Origin, X-Requested-With, Content-Type,  Accept");
   next();
   });

   app.use((req, res, next) => {
   res.send('Welcome to Express');
   });

   app.listen(port, () => {
   console.log(`Server running on port ${port}`)
   });  

- I'll start my server to see if it works and running on port 5000, which is the port specified in my code inputed in index.js file via v.i editor above. Run :

  `node index.js`    

  ![node index js running](https://user-images.githubusercontent.com/107736487/174439750-8273ff92-1d07-49b4-88eb-4332d7474e74.PNG)

- Now i'll need to edit an inbound rule in my EC2 Security Groups to open port 5000.

  ![5000 port](https://user-images.githubusercontent.com/107736487/174439767-e7b90430-1e2a-4dd1-91d5-e9e39244a06a.PNG)

- Open up your browser and try to access your server’s Public IP or Public DNS name followed by port 5000:

  `http://18.168.204.32:5000`
   
   ![express](https://user-images.githubusercontent.com/107736487/174439779-0bee2c40-2d53-4bfa-8dc2-0c23ed2553eb.PNG)

- **Routes**
---

  There are three actions that my To-Do application needs to be able to do:

  1. Create a new task
  2. Display list of all tasks
  3. Delete a completed task

  Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE.

  For each task, i need to create *routes* that will define various endpoints that the *To-do* app will depend on. 
  
- First. i'll create a folder routes

  `mkdir routes`

- Change directory to routes folder.

  `cd routes`

- Now, i'll create a file **api.js** with the command below:

  `touch api.js`

- Open the file with the command below

  `vim api.js`

- Copy below code in the file

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
- **Models**
---

  Since the app is going to make use of **Mongodb** which is a NoSQL database, i'll need to create a model.

  I'll use models to define the database schema.

  The Schema is a blueprint of how the database will be constructed, including other data fields that may not be required to be stored in the database.

- To create a Schema and a model, install **mongoose** which is a Node.js package that makes working with mongodb easier.

  `npm install mongoose`

- Create a new folder models :

  `mkdir models`

- Change directory into the newly created ‘models’ folder with

  `cd models`

- Inside the models folder, create a file and name it todo.js

  `touch todo.js`  

-  Open the file with the command below

   `vim todo.js`

- Copy below code in the file

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
![](./Images3/cat%20todo.js)

- Now i'll need to update my routes from the file **api.js** in ‘routes’ directory to make use of the new model.

  In Routes directory, i'll open api.js with vim api.js, delete the code inside with :%d command and paste the code below into it then save and exit

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
- **MONGODB DATABASE**
---
- I need a database where i'll store my data. For this i will make use of mLab. 

- I'll create a file in my **Todo** directory and name it **.env**.

  `touch .env`

- I'll add the connection string of my cloud mongoDB to access the database in it, I'll copy and past the string into .env file using vi editor
  ```
  mongodb+srv://kris:<password>@cluster0.4kq6s.mongodb.net/<Dbname>?retryWrites=true&w=majority
  ```
  Note i will Replace **password** with the password for the **kris** user 

  ![connect MongoDB cluster](https://user-images.githubusercontent.com/107736487/174439935-965560e4-d274-43e1-b6bc-3abf47349e0f.PNG)

- Now i'll need to update the **index.js** using Vim editor to reflect the use of **.env** so that **Node.js** can connect to the database.

  i'll delete existing content in the file, and update it with the entire code below.   
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
  res.header("Access-Control-Allow-Origin",  "\*");
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
 - Start my server using the command to see if Database is connected successfully:

   `node index.js` 

   ![database connected successfully](https://user-images.githubusercontent.com/107736487/174439986-0b3a781f-9fa2-4531-9d1b-26d230da0641.PNG)

- Testing Backend Code without Frontend using RESTful API

  Using Postman, i'll test all the API endpoints and make sure they are working

  1. I'll open my Postman, create a **POST request** to the API http://18.168.203.73:5000/api/todos. This request sends a new task to my To-Do list so the application could store it in the database.

     ![Post 1](https://user-images.githubusercontent.com/107736487/174440032-cc437beb-eb56-4a66-b8c2-67f7230333c3.PNG)
     ![post body](https://user-images.githubusercontent.com/107736487/174440065-5093ccca-bf83-4ea8-8391-f5c1a8b03943.PNG)

  2. I'll Create a **GET request** to my API on http://18.168.203.73:5000/api/todos. This request retrieves all existing records from out To-do application (backend requests these records from the database and sends it us back as a response to GET request).
 
     ![get](https://user-images.githubusercontent.com/107736487/174440199-4ac1b771-3c3d-4cd4-9198-d32bf9000c9e.PNG)

## STEP 2 – FRONTEND CREATION     
---
- In Todo Directory run `npx create-react-app client` which will create a new folder in my Todo directory called client, where i will add all the react code

  `npx create-react-app client`

- #### **Running a React App**

  Before testing the react app, there are some dependencies that need to be installed.

  1. **Install concurrently**. It is used to run more than one command simultaneously from the same terminal window.

   
     `npm install concurrently --save-dev
     `

  2. **Install nodemon**. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.

      `npm install nodemon --save-dev
      `
  3. In my Todo folder i'll open the **package.json** file. I will edit and remove the script section and replace with the code below.
     ```
     "scripts": {
     "start": "node index.js",
     "start-watch": "nodemon index.js",
     "dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
     },
     ```
- **Configure Proxy in package.json**

   1. Change directory to ‘client’

      `cd client`

    2. Open the package.json file

       `vi package.json`

    3. Add the key value pair in the package.json file "proxy": "http://localhost:5000".

    ![proxy 2](https://user-images.githubusercontent.com/107736487/174440239-c3ffe3db-9af5-41f3-bfa3-4a4d7f31f084.PNG)

  Now, ensure you are inside the Todo directory, and simply do:

  `npm run dev`     

- I can see that my app is open and running on localhost:3000  

   ![port 3000 running](https://user-images.githubusercontent.com/107736487/174440344-3f1cb14a-12e9-4180-a22a-426df783ed27.PNG)

- In order to be able to access the application from the Internet i have to open TCP port 3000 on my EC2 instance by adding a new Security rule

   ![ec2 3000](https://user-images.githubusercontent.com/107736487/174440373-0646daad-576e-49de-96d0-ca6e1049918a.PNG)

- React App opening successfuly on port 3000

  ![react app](https://user-images.githubusercontent.com/107736487/174440561-381dcef6-a209-4da0-a60d-c97efebd2183.PNG)

- #### **Creating my React Components**
  From my Todo directory run

  `cd client`

  move to the src directory

  `cd src`

  Inside my src folder i'll create another folder called **components**

  `mkdir components`

  Move to the components directory

   `cd components`

  Inside ‘components’ directory create three files **Input.js**, **ListTodo.js** and **Todo.js**.

  `touch Input.js ListTodo.js Todo.js`

  Open Input.js file

  `vi Input.js`

  Copy and paste the following:
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
  <button onClick={this.addTodo}>add todo</  button>
  </div>
  )
  }
  }

  export default Input
  ```

- To make use of **Axios**, which is a Promise based HTTP client for the browser and node.js, i'll need to cd into my client from my terminal and run `yarn add axios` or `npm install axios`.

- cd into client directory

  `cd ../..`

- Install Axios

  `npm install axios` 

- Go to **‘components’** directory

  `cd src/components`

- Open ListTodo.js

  `vi ListTodo.js`

  in the **ListTodo.js,** copy and paste the following code: 
  ```
  import React from 'react';

  const ListTodo = ({ todos, deleteTodo }) => {

  return (<ul>
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
- Then in my **Todo.js** file i'll copy and paste the following code into it. 
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
- I'll need to make little adjustment to my react code. Delete the logo and adjust my **App.js** to look like this.

- Move to the **src** folder

  ` cd ..` 

- Using vi editor, go to **App.js** file

  `vi App.js` 

- Copy and paste the code below into **App.js**   
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

- Using vi editor, go to **App.css** file

  `vi App.css` 

- Copy and paste the code below into **App.css**   
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
-  Using vi editor, go to **index.css** file

    `vi index.css` 

- Copy and paste the code below into **index.css**   
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
- Go to the Todo directory

  `cd ../..`

- In the Todo directory run: `npm run dev` to confirm that my To-Do app is functioning properly

  `npm run dev`  

  ![my todo](https://user-images.githubusercontent.com/107736487/174441430-947d51e0-7328-416c-8f03-b1dd0e19fb55.PNG)

- End of Project. My To-Do App can create a task, delete a task and view all my tasks.

  ![end of pro](https://user-images.githubusercontent.com/107736487/174441459-a9a78c88-426f-4393-a7aa-46a2f61db291.PNG)

