# Modern MERN Stack Implementation & Multi-Tier Deployment

This repository contains a production-ready, fully decoupled MERN (MongoDB, Express, React, Node.js) application architecture. The project demonstrates the separation of frontend state management from backend RESTful API services, structured specifically for multi-tier cloud infrastructure.

## 🛠️ Tech Stack & Boundary Layering
* **Edge Layer:** Dynamic Single Page Application (SPA) powered by React.js.
* **Application API Layer:** Stateless routing engine built with Express.js and Node.js.
* **Data Persistence Layer:** Schema-validated cloud storage via MongoDB Atlas and Mongoose ODM.

---

## 📂 Repository Blueprint
```
.
├── models/             # Strict Mongoose schema models (Data validation)
├── routes/             # Isolated Express endpoint routers (API layer)
├── client/             # Decoupled React production source files
│   └── src/
│       └── components/ # Stateful UI Components (Input.js, ListTodo.js, Todo.js)
├── index.js            # Node/Express runtime entry point
└── .env                # Runtime environment context (Externalized credentials)
```

## ⚙️ Core Technical Implementation
​1\. Schema Enforcement & Data Flow (MongoDB/Mongoose)
* ​Modeled asynchronous data boundaries using Mongoose to enforce validation rules on a schemaless database.
* ​Isolated operational queries from the web server layer to ensure data sanitization.

```
// Example Schema (models/todo.js)
const mongoose = require('mongoose');
const TodoSchema = new mongoose.Schema({
  action: { type: String, required: [true, 'The todo text field is required'] }
module.exports = mongoose.model('todo', TodoSchema);
```

2\. Stateless API Architecture (Node.js/Express)
* ​Developed decoupled routing files to expose clean HTTP interfaces (GET, POST, DELETE).
​
* Secured the application environment by externalizing database connection strings using .env injections.

```
// Example API Route (routes/api.js)
router.get('/todos', (req, res, next) => {
  Todo.find({}, 'action')
    .then(data => res.json(data))
    .catch(next);
});
```

3\. Stateful Asynchronous Frontend (React)
​Engineered controlled UI input structures to guarantee predictable application states during user interaction.
​Implemented Axios lifecycle triggers to bind the browser UI state directly to remote API database responses.

```
// Example Component Lifecycle (client/src/components/Todo.js)
componentDidMount() {
  this.getTodos();
}
getTodos = () => {
  axios.get('/api/todos')
    .then(res => { if (res.data) { this.setState({ todos: res.data }) } })
    .catch(err => console.log(err));
}
```

4\. Cross-Port Proxy Configuration
* ​Established a localized package-level reverse proxy layer to route client traffic smoothly across the port boundaries, entirely eliminating CORS exceptions.

```
// client/package.json
"proxy": "http://localhost:5000"
```

## 💡 Engineering Key Takeaway
​Developing this MERN stack architecture highlights the power of decoupling the presentation layer from backend logic. Separating the code into autonomous frontend and backend services provides the identical foundation required for deploying cloud-native containerized environments (Docker/Kubernetes) and automated CI/CD configurations.
