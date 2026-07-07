# MERN web stack implementation in AWS project 3

The MERN stack is a collection of technologies that help developers build robust and scalable web applications using JavaScript. The acronym “MERN” stands for MongoDB, Express, React, and Node. js, with each component playing a role in the development process.

## Step 0 - Preparing prerequisites

In order to complete this project you will need an AWS account and a virtual server with Ubuntu Server OS.


1\. Log to aws account console and create EC2 instance of t2.micro type with Ubuntu 24.04 LTS (HVM) Server launch in the default region us-east-1.

![Screenshot](images/creating-instance.png)

2\. Create new key pair

![Screenshot](images/create-key-pair.png)

3\. Network setting create new security group or use existing security group

![Screenshot](images/security-group.png)


4\. View Instance

![Screenshot](images/view-instance.png)

5\. Instance Details
![Screenshot](images/instance-details.png)


6\. Configure security group with the following inbound rules:

Allow traffic on port 22 (SSH) with source from any IP address. This is opened by default.
Allow traffic on port 80 (HTTP) with source from anywhere on the internet.
Allow traffic on port 443 (HTTPS) with source from anywhere on the internet.
Allow traffic on port 5000 (Custom) with source from anywhere on the internet.

![Screenshot](images/security-rules.png)


7\. Give Permission for the private ssh key that we used either created or existing during launching instance and downloaded

```
chmod 400 <path-to-your-key-file.pem>
```

Then connect the instance using instance Public Ip Addresss or domain name

ssh -i <path-to-your-key-file.pem> ubuntu@<your-instance-public-ip-or-dns>

![Screenshot](images/connecting-server.png)


## Step 1 - Backend configuration

1\. Update and upgrade the server's package index 

```
sudo apt update && sudo apt upgrade -y
```

2\. Get the location of Node.js software from Ubuntu repositories

```
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```

![Screenshot](images/nodejs-download.png)

3\. Install Node.js on the server 

```
sudo apt install -y nodejs
```

![Screenshot](images/install-nodejs.png)


4\. Verify the Node installation with the command below.

```
node -v
```
This gives the node version

## Application Code Setup
---

Create a new directory for the To-Do project:

![Screenshot](images/packagejson-new.png)


This is to initialize the project directory and in the process, creates a new file called package.json. This file will contain information about your application and the dependencies it needs to run. Follow the prompts after running the command. You can press "Enter" several times to accept default values, then accept to write out the package.json file by typing yes.

## Install ExpressJs

Express is a framework for Node.js. It simplifies development and abstracts a lot of low level details. For example, express helps to define routes of your application based on HTTP methods and URLs.

1\. Install Express using: 

```
npm install express
```

2\. Create a file index.js and run ls to confirm the file 

```
touch index.js
```
```
ls
```
3\. Install dotenv module. It is a popular Node.js library used for loading environment variables from a .env file into process.env

```
npm install dotenv
```

![Screenshot](images/install-express.png)



5\. Open the index.js file using your editor vim or nano

```
vim index.js
```

6\. Type the code below into it and save.

```
const express = require('express'); require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => { res.header("Access-Control-Allow-Origin", "*"); res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept"); next(); });

app.use((req, res, next) => { res.send('Welcome to Express'); });

app.listen(port, () => { console.log(Server running on port ${port}); });

```

Notice that we have specified to use port 5000 in the code. This will be required later when we go on the browser. But it was already added when creating the instance

7\. Start the server to see if it works. Open your terminal in the same directory as your index.js file. Run:

```
node index.js
```

![Screenshot](images/server-running.png)


8\. Open up your browser and try to access your server's Public IP or Public DNS name followed by port 5000:
 

Access the server with the public IP followed by the port http://100.53.2.33:5000

![Screenshot](images/browser-view.png)


## Routes

There are three actions that our To-Do application needs to be able to do:

* Create a new task

* Display list of all tasks

* Delete a completed task

1\. Create a folder and name it 'routes':

```
mkdir routes
```

Switch to 'routes' directory and create a file api.js.

```
touch api.js 
```

```
vim api.js
```

2\. Copy the code below into the file const express = require('express'); const router = express.Router();


```
const express = require('express'); const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

});

module.exports = router;


```

## Models

A model is at the heart of JavaScript based applications and it is what makes it interactive. Models was used to define the database schema. This is important in order be able to define the fields stored in each Mongodb document. In essence, the schema is a blueprint of how the database is constructed, including other data fields that may not be required to be stored in the database. These are known as virtual properties. To create a schema and a model, mongoose was installed, which is a Node.js package that makes working with mongodb easier.

Change the directory back to 'Todo' folder and install mongoose 

```
npm install mongoose
```

![Screenshot](images/install-mongoose.png)

Create a new folder models, switch to models directory, create a file todo.js inside models. Open the file:

```
mkdir models && cd models && touch todo.js && vim todo.js
```

Paste the code below into the file: 
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


![Screenshot](images/todojs.png)

The routes was updated from the file api.js in the 'routes' directory to make use of the new model. 

3. In Routes directory, open api.js and delete the code inside with :%d:

```
vim api.js
```

Paste the new code below into it 

```
const express = require('express'); 
const router = express.Router(); 
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {
 // This will return all the data, exposing only the id and action field to the client
    Todo.find({}, 'action')
        .then(data => res.json(data))
        .catch(next);
});

router.post('/todos', (req, res, next) => {
    if (req.body.action) {
        Todo.create(req.body) 
            .then(data => res.json(data)) .catch(next);
    } else {
        res.json({ error: "The input field is empty" });
    }
});

router.delete('/todos/:id', (req, res, next) => {
    Todo.findOneAndDelete({"_id": req.params.id})
        .then(data => res.json(data))
        .catch(next);
});

module.exports = router;
```


![Screenshot](images/api.png)

## MongoDB Database

Create a MongoDB database and collection inside mLab AWS cloud provider, in region N. Virginia (us-east-1) was selected. MongoDB Cluster Overview


Allow access to the MongoDB database from anywhere (Not secure, but it is ideal for testing) and change the time of deleting the entry from 6 Hours to 1 Week.

![Screenshot](images/cluster-overview.png)

![Screenshot](images/ip-access.png)

Create database named 'to-do-database' and collections named 'todos'.

![Screenshot](images/database-user.png)

To get the connection string:

![Screenshot](images/connect-cluster.png)

Copy the connection string.

Create a file in your Todo directory and name it .env:

```
touch .env && vim .env
```
Here is my string:

```
mongodb+srv://to-do-database:<db_password>@cluster0.byqr3ft.mongodb.net/?appName=Cluster0
```
Replace with the password for your user

```
mongodb+srv://to-do-database:aYomikun@2sam@cluster0.byqr3ft.mongodb.net/?appName=Cluster0
```

Update the index.js to reflect the use of .env so that Node.js can connect to the database.

Simply delete existing content in the file, and update it with the entire code below.

```
vim index.js
```
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

![Screenshot](images/update-indexjs.png)

Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file.

Start your server using the command 

```
node index.js
```

![Screenshot](images/server-runnin.png)

## Testing Backend Code without Frontend using RESTful API

So far we have written backend part of our To-Do application, and configured a database, but we do not have a frontend UI yet. We need ReactJS code to achieve that. But during development, we will need a way to test our code using RESTfulL API. Therefore, we will need to make use of some API development client to test our code.

In this project, we will use Postman to test our API.

How to Install Postman In our machine

1\. Download Postman: Go to the Postman website and download the version of Postman that is compatible with your operating system (Windows, macOS, or Linux).

2\. Install Postman: Once the download is complete, run the installer file. Follow the on-screen instructions to complete the installation process.

3\. Launch Postman: After installation, you can launch Postman by finding it in your list of installed applications or by searching for "Postman" in the search bar of your operating system.

4\. Sign in or Create an Account (Optional): While it's not required, signing in or creating a Postman account can unlock additional features and capabilities, such as cloud synchronization of your collections and environments.

5\. Start Using Postman: Once Postman is launched, you can start using it to create and send HTTP requests, test APIs, and collaborate with your team on API development projects.

Test all the API endpoints and make sure they are working. For the endpoints that require body, you should send JSON back with the necessary fields since it’s what we setup in our code.

Now open your Postman, create a POST request to the API http://ip:5000/api/todos. This request sends a new task to our To-Do list so the application could store it in the database.

Note: make sure your set header key Content-Type as application/json

Create POST requests to the API

![Screenshot](images/post-start.png)

Check Database Collections

![Screenshot](images/database-collection.png)

Make a GET requests to the API This request retrieves all existing records from our To-Do application (backend requests these records from the database and sends us back as a response to GET request).

![Screenshot](images/get-request-all.png)

Create a DELETE requests to the API

![Screenshot](images/delete-request.png)



## Step 2 - Frontend Creation

With all functionalities configured from the backend, a user interface will be created for a Web client (browser) to interact with the application via API

In the same root directory as your backend code, which is the Todo directory, run:

```
npx create-react-app client
```

![Screenshot](images/install-react.png)


This created a new folder in the Todo directory called client, where all the react code was added.

### Running a React App

1\. Before testing the react app, the following dependencies needs to be installed in the project root directory. Install concurrently. It is used to run more than one command simultaneously from the same terminal window. 

```
npm install concurrently --save-dev
```

![Screenshot](images/install-concurrently.png)

2\. Install nodemon used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.

```
npm install nodemon --save-dev
```

3\. In Todo folder open the package.json file, change the highlighted part of the below screenshot and replace with the code below: 

```
"scripts": { "start": "node index.js", "start-watch": "nodemon index.js", "dev": "concurrently "npm run start-watch" "cd client && npm start"" }
```

### Configure Proxy in package.json

1\. Change directory to client and open the package.json file

```
cd client && vim package.json
```
2\. Add the key value pair in the package.json file:

```
"proxy": "http://localhost:5000"
```

![Screenshot](images/proxy-key-pair.png)


The purpose of adding the proxy configuration is to make it possible to access the application directly from the browser by simply calling the server url like http://localhost:5000 rather than always including the entire path like http://localhost:5000/api/todos

Ensure you are inside the Todo directory, and simply do: 

```
cd ..
```
```
npm run dev
```

Important note: In order to be able to access the application from the Internet you have to open TCP port 3000 on EC2 by adding a new Security Group rule, but this has been added to the security group when the instance was created. 

![Screenshot](images/update-security-group.png)


![Screenshot](images/webpack-compilled.png)

The app opened and started running on localhost:3000

### Creating React Components

One of the advantages of react is that it makes use of components, which are reusable and also makes code modular. For the Todo app, there are two stateful components and one stateless component. From Todo directory, run:

```
cd client
```

 Move to the "src" directory 
 
 ```
 cd src
 
```

2\. Inside your src folder, create another folder called "components", Inside the 'components' directory create three files "Input.js", "ListTodo.js" and "Todo.js". 

```
mkdir components
```

```
cd components
``` 
```
touch Input.js ListTodo.js Todo.js
```

![screenshot](images/components.png)


Open Input.js file vim Input.js Paste in the following: 

```
import React, { Component } from 'react';

import axios from 'axios';

class Input extends Component { 
   state = {
     action: "" 
   }

   handleChange = (event) => {
           this.setState({ action: event.target.value });
   }

   addTodo = () => {
       const task = { action: this.state.action };

       if (task.action && task.action.length > 0) {
            axios.post('/api/todos', task)
                    .then(res =>{
                        if (res.data) {
                                this.props.getTodos();
                                this.setState({ action: "" });
                        }
                    }) .catch(err => console.log(err));
       } else {
               console.log('Input field required');
       }
   }

   render() {
       let { action } = this.state;
       return (
           <div>
               <input type="text" onChange={this.handleChange} value={action} />
               <button onClick={this.addTodo}>add todo</button> 
           </div>
       );
   }
}

export default Input
```

![screenshot](images/inputjs.png)

In oder to make use of Axios, which is a Promise based HTTP client for the browser and node.js, you need to cd into your client from your terminal and run yarn add axios or npm install axios. Move to the client folder and install Axios:

```
npm install axios
```

![screenshot](images/install-axios.png)

Go to components directory 

```
cd src/components
```

After that open the ListTodo.js 

```
vim ListTodo.js
```
Copy and paste the following code:

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

![screenshot](images/list-todojs.png)

Then in the Todo.js file, write the following code vim Todo.js

```
import React, { Component } from 'react'; import axios from 'axios';

import Input from './Input'; import ListTodo from './ListTodo';

class Todo extends Component { state = { todos: [] };
componentDidMount() { this.getTodos(); }

getTodos = () => { axios.get('/api/todos') .then(res => { if (res.data) { this.setState({ todos: res.data }); } }) .catch(err => console.log(err));
}

deleteTodo = (id) => { axios.delete(`/api/todos/${id}`) .then(res => { if (res.data) { this.getTodos(); } }) .catch(err => console.log(err));
}

render() {
  let { todos } = this.state;

  return (
    <div>
      <h1>My Todo(s)</h1>
      <Input getTodos={this.getTodos} />
      <ListTodo
        todos={todos}
        deleteTodo={this.deleteTodo}
      />
    </div>
  );
}
}
export default Todo;
```

![screenshot](images/Todo.js.png)

We need to make a little adjustment to our react code. Delete the logo and adjust our App.js to look like this. Move to src folder:

```
cd ..
```

Ensure to be in the src folder and run:

```
vim App.js
```

Copy and paste the following code:

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
};
export default App;
```

![screenshot](images/Appjs-edit.png)

In the src directory, open the App.css:

```
vim App.css
```

Paste the following code into it:

```
.App { text-align: center; font-size: calc(10px + 2vmin); width: 60%; margin-left: auto; margin-right: auto; }

input { height: 40px; width: 50%; border: none; border-bottom: 2px #101113 solid; background: none; font-size: 1.5rem; color: #787a80; }

input:focus { outline: none; }

button { width: 25%; height: 45px; border: none; margin-left: 10px; font-size: 25px; background: #101113; border-radius: 5px; color: #787a80; cursor: pointer; }

button:focus { outline: none; }

ul { list-style: none; text-align: left; padding: 15px; background: #171a1f; border-radius: 5px; }

li { padding: 15px; font-size: 1.5rem; margin-bottom: 15px; background: #282c34; border-radius: 5px; overflow-wrap: break-word; cursor: pointer; }

@media only screen and (min-width: 300px) { .App { width: 80%; }

input { width: 100%; }

button { width: 100%; margin-top: 15px; margin-left: 0; } }

@media only screen and (min-width: 640px) { .App { width: 60%; }

input { width: 50%; }

button { width: 30%; margin-left: 10px; margin-top: 0; } }

```
![screenshot](images/Appcss-edit.png)

In the src directory, open the index.css:

```
vim index.css
```

Copy and paste the code below:

```
body {
  margin: 0;
  padding: 0;
  font-family: sans-serif;
  background-color: #2b2b2b;
  color: #bfbfbf;
}
.Todo {
  max-width: 500px;
  margin: 60px auto;
  padding: 20px;
  background: #1f1f1f;
  border-radius: 5px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

input {
  width: 70%;
  padding: 10px;
  border: 1px solid #444;
  background: #333;
  color: #fff;
}
button {
  width: 25%;
  padding: 10px;
  background: #444;
  color: #fff;
  border: none;
  cursor: pointer;
}
```

![screenshot](images/indexcss-edit.png)

![screenshot](images/index-css.png)

Go to the Todo directory:

```
cd ../..
```

Run:

```
npm run dev
```

![screenshot](images/app-fully-compiled.png)

At this point, the To-Do app is ready and fully functional with the functionality discussed earlier: Creating a task, deleting a task, and viewing all the tasks. The client can now be viewed in the browser

Add some todos via the browser.

![screenshot](images/add-todo-task.png)

Check on the MongoDB database

![screenshot](images/mongodb-display.png)

### Conclusion

This project demonstrates the successful implementation of a complete MERN stack application hosted on AWS. By integrating MongoDB for data management, Express.js and Node.js for backend services, and React.js for the frontend interface, a responsive and efficient To-Do application was developed and deployed. The implementation highlights key concepts such as RESTful API development, database connectivity, environment variable management, React component architecture, and cloud deployment. Following the procedures outlined in this guide provides a solid foundation for building, deploying, and managing scalable full-stack web applications using the MERN stack on AWS infrastructure.