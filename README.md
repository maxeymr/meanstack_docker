# MEAN Stack with Docker

## Setting up an image for code development. This documents the step-by-step process from the initial installation to the final production

### Setup Clients

#### 1. Make sure Docker, Docker Compose, and Node are installed. In a terminal, run the following commands to verify

```
docker -v
Returned Docker version 18.09.7, build 2d0083d
docker-compose -v
Returned docker-compose version 1.8.0, build unknown
node -v
Returned v12.18.2
npx -v
Returned 6.14.5
npm -v
Returned 6.14.5
```

Note:
" Docker Compose required install with the command docker sudo apt install docker-compose
" Node with npm/npx required install with the following commands sudo apt install docker-compose and sudo apt-get install nodejs

#### 2. At your home directory, run the command cd meanstack to create a directory to work from.

3. Setup MongoDB
   This only verifies the commands to pull a docker image from Docker Hub  to understand how it can work.  
   Run the command docker pull mongo to get the latest mongodb image from Docker Hub.

4. Setup Express API Application
   In the meanstack directory, create a directory with the command mkdir express

In the express directory, create a file named package.json with the following text

```JSON
{
"name": "express",
"version": "0.0.0",
"private": true,
"scripts": {
"start": "node server.js"
},
"dependencies": {
"body-parser": "~1.15.2",
"express": "~4.14.0",
"mongoose": "^4.7.0"
}
}
```

In the express directory, create a file named server.js with the following text

```JavaScript
//Get dependencies
const express = require('express');
const path = require('path');
const http = require('http');
const bodyParser = require('body-parser');

//Get API routes
const api = require('./routes/api');

const app = express();

//Parsers for POST data
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));

// Cross Origin middleware
app.use(function(req, res, next) {
res.header("Access-Control-Allow-Origin", "\*")
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept")
next()
})

//Set our api routes
app.use('/', api);

//Get port from environment and store in Express
const port = process.env.PORT || '3000';
app.set('port', port);

//Create HTTP server
const server = http.createServer(app);

//Listen on provided port to network interfaces
server.listen(port, () => console.log(`API running on localhost:${port}`));
```

In the express directory, create a routes directory with an api.js file containing the following text

```JavaScript
//Import dependencies
const mongoose = require('mongoose');
const express = require('express');
const router = express.Router();

//MongoDB URL from the docker-compose file
const dbHost = "mongodb://localhost:27017/meanstack";

// Connect to mongodb
mongoose.connect(dbHost);

// create mongoose schema
const todoSchema = new mongoose.Schema({
tag: String,
task: String

},
{ timestamps: true });

// create mongoose model
const Todo = mongoose.model('Todo', todoSchema);

/_ GET api listing _/
router.get('/', (req, res) => {

    res.send('Todo APIs...');

});

/_ GET api listing _/
router.get('/health', (req, res) => {
res.status(200).json({"Status": "Up"});
});

/_ GET all todo items _/
router.get('/todos', (req, res) => {
Todo.find({}, (err, todos) => {
if (err) res.status(500).send(error)
res.status(200).json(todos);
});
});

/_ GET items by id _/
router.get('/todos/id/:id', (req, res) => {
console.log('\_id=%s', req.params.id);
Todo.find({'\_id': req.params.id}, (err, todos) => {
//Todo.findById(req.params.id, (err, todos) => {
if (err) res.status(500).send(error)
res.status(200).json(todos);
});
});

/_ GET items by tag _/
router.get('/todos/tag/:tag', (req, res) => {
console.log('Tag=%s', req.params.tag);
Todo.find({'tag': req.params.tag}, (err, todos) => {
if (err) res.status(500).send(error)
res.status(200).json(todos);
});
});

/_ Create a todo item _/
router.post('/todos', (req, res) => {
let todo = new Todo({
tag: req.body.tag,
task: req.body.task
});
todo.save(error => {
if (error) res.status(500).send(error);
res.status(201).json({
message: 'Todo item created successfully'
});
});
});

module.exports = router;
```

Run the following commands to verify your API app

```
npm install
npm start
```

The application will be running at http://localhost:3000/ displaying the text "Todo APIs" if it is working.
Other endpoints created of interest by the curl command

```
curl http://localhost:3000/health
curl -d '{"tag":"Food", "task":"Get groceries"}' -H "Content-Type: application/json" -X POST http://localhost:3000/todos
curl http://localhost:3000/todos
curl http://localhost:3000/todos/tag/Food
curl http://localhost:3000/todos/id/[needsid]
```

5. Setup Angular Application
   In the meanstack directory, create a directory with the command mkdir angular

Run the following command to setup an angular app and set up dependencies in the angular directory

```
npx @angular/cli new angular
```

Update the angular/package.json file start script to

```
ng serve --host 0.0.0.0
```

In the angular/src directory, delete the file named app_modules.ts

In the angular/src/app directory, delete the file named app.component.spec.ts

In the angular/src directory, update the index.html file to the following

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Angular Client</title>
    <base href="/" />

    <meta name="viewport" content="width=device-width, initial-scale=1" />

    <!-- Bootstrap CDN -->
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.2/css/bootstrap.min.css" />

    <link rel="icon" type="image/x-icon" href="favicon.ico" />
  </head>
  <body>
    <app-root>Loading...</app-root>
  </body>
</html>
```

In the angular/src/app directory, update the app.component.html file to the following

```html
<nav class="navbar navbar-light bg-faded">
  <a class="navbar-brand" href="#">M.E.A.N. Stack</a>
</nav>

<div class="container">
  <div class="row">
    <div class="col">
      <h4>Add new todo item...</h4>
      <form>
        <div class="form-group">
          <label for="tag">Tag</label>
          <input type="text" class="form-control" id="tag" #tag />
        </div>
        <div class="form-group">
          <label for="task">Task</label>
          <input type="text" class="form-control" id="task" #task />
        </div>
        <button type="button" (click)="addTodo(tag.value, task.value)" class="btn btn-primary">Add todo</button>
      </form>
    </div>
  </div>
  <br />
  <br />
  <div class="row">
    <div class="col">
      <h4>Todos...</h4>
      <ul class="list-group" *ngFor="let todo of todos; let i = index">
        <li class="list-group-item">{{i+1}}. <b>{{todo.tag}}</b> | <i>{{todo.task}}</i></li>
      </ul>
    </div>
  </div>
</div>
```

In the angular/src/app directory, update the app.component.ts file to the following

```TypeScript
import { Component, OnInit } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Component({
selector: 'app-root',
templateUrl: './app.component.html',
styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
title = 'app works!';

// Link to our api, pointing to localhost
API = 'http://localhost:3000';

// Declare empty list of todos
todos: any[] = [];

constructor(private http: HttpClient) {}

// Angular 2 Life Cycle event when component has been initialized
ngOnInit() {
this.getAllTodos();
}

// Add one todo to the API
addTodo(tag, task) {
this.http.post(`${this.API}/todos`, {tag, task})
.subscribe(() => {
this.getAllTodos();
})
}

// Get all todos from the API
getAllTodos() {
this.http.get(`${this.API}/todos`)
.subscribe((todos: any) => {
console.log(todos)
this.todos = todos
})
}
}
```

In the angular/src/app directory, update the app.module.ts file to the following

```Typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { HttpClientModule } from '@angular/common/http'; // add http client module

import { AppComponent } from './app.component';

@NgModule({
declarations: [
AppComponent
],
imports: [
BrowserModule,
HttpClientModule // import http client module
],
providers: [],
bootstrap: [AppComponent]
})
export class AppModule { }
```

Run the following commands to verify your API app

```
npm install
npm start
```

### The application will be running at http://localhost:4200/ displaying the MEAN Stack Todo app if it is working. You would need the mongodb and express servers running locally for the entire app to work at this point.

### Run the Angular application successfully in the Docker container

### Setup container images

1. Run MongoDB image locally (to see how it runs)
   Run the mongo container with the command: docker run --rm -d --name mongodb -p 27017:27017 mongo
   It may take a minute or more for the container to complete connection but check the logs several times by running the following command (to see when the container is running): docker logs mongodb
   Go to the url http://localhost:27017/ - This url will show a message of It looks like you are trying to access MongoDB over HTTP on the native driver port. when mongo is running.

2. Create and run image for Express
   In the express directory, create a file named Dockerfile, with the following text
   #Docker file instructions...

```Dockerfile
# create image
# set directory for app
# copy dependencies
# install dependencies
# copy code
# expose port
# serve app

FROM node:6
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
COPY package.json /usr/src/app
RUN npm install
COPY . /usr/src/app
EXPOSE 3000
CMD ["npm", "start"]
```

Extra: You can setup a .dockerignore file so certain files or folders will not copy to the app folder (for the Dockerfile).
In the express directory, create a file named .dockerignore, with the following text

```
node_modules/ The ignore file may be hidden so you will have to view hidden files.
```

Build the docker image for express:dev from a terminal (make sure to cd to the express folder first) with the following command

```
docker build --rm -t express:dev .
```

You can test the image out by running the command: docker run --rm -d --name express -p 3000:3000 express:dev
It may take a minute or more for the container to complete connection but check the logs several times by running the following command (to see when the container is running): docker logs express

3. Test Express endpoints
   The app will be running at: http://localhost:3000/

http://localhost:3000/health returns back a {status:up} detail.

Other endpoints for testing in curl

```
curl http://localhost:3000/
curl -d '{"tag":"Food", "task":"Get groceries"}' -H "Content-Type: application/json" -X POST http://localhost:3000/todos
curl http://localhost:3000/todos
curl http://localhost:3000/todos/tag/Food
curl http://localhost:3000/todos/id/[needsid]
```

4. Create and run image for Angular
   In the angular directory, create a file named Dockerfile, with the following text
   #Docker file instructions...

```Dockerfile
# create image
# set directory for app
# copy dependencies
# install dependencies
# copy code
# expose port
# serve app

FROM node:10
RUN mkdir -p /app
WORKDIR /app
COPY package\*.json /app/
RUN npm install
COPY . /app/
EXPOSE 4200
CMD ["npm","start"]
```

Extra: You can setup a .dockerignore file so certain files or folders will not copy to the app folder (for the Dockerfile).
In the angular directory, create a file named .dockerignore, with the following text

```
node_modules/ The ignore file may be hidden so you will have to view hidden files.
```

Build the docker image for angular:dev from a terminal (make sure to cd to the angular folder first) with the following command

```
docker build --rm -t angular:dev .
```

You can test the image out by running the command: docker run --rm -d --name angular -p 4200:4200 angular:dev
It may take a minute or more for the container to complete connection but check the logs several times by running the following command (to see when the container is running with a Compiled Successfully message): docker logs angular

5. Test Angular application and verify it is running
   The angular app will be running at: http://localhost:4200/
   You will be able to add Todo items if the mongodb and express containers are running.

Angular app running in a browser with 1 todo item added

6. Stop the running containers with the commands

```
docker stop mongodb
docker stop express
docker stop angular
```

### Use Docker Compose to manage the Angular application running inside the Docker container

### Push images to Docker Hub and run with Docker Compose

1. Setup Docker Compose
   From your root home directory, cd to the meanstack directory: cd meanstack

Run the following command to create a docker compose yml file: touch docker-compose.yml and add the following text to the file

```yml
version: '2' #docker-compose version

#Define the services and containers
services:
database: #Service name
image: mongo #Image to build container from
ports: - "27017:27017"

express: #Service name
image: maxeymr/meanstack_express:dev #Image to build container from
ports: - "3000:3000"
links: - database

angular: #Service name
image: maxeymr/meanstack_angular:dev #Image to build container from
ports: - "4200:4200"
volumes: - ./angular:/app #Trigger recompilation in container
```

2. Push express image to Docker Hub (you will have to be logged in to docker and have an account)
   To prepare the connection to the mongodb, in the express directory and then in the routes directory, modify the api.js. Change the const dbHost to

```javascript
const dbHost = "mongodb://database/meanstack";
```

Change to the express directory with the command: cd meanstack/express
Build the docker image to post to docker hub with the command:

```
docker build --rm -t maxeymr/meanstack_express:dev .
```

Push the docker image to docker hub with the command: docker push maxeymr/meanstack_express:dev

Verify the new repository is in docker hub.

3. Push angular image to Docker Hub (you will have to be logged in to docker and have an account)

Change to the express directory with the command: cd meanstack/angular
Build the docker image to post to docker hub with the command:

```
docker build --rm -t maxeymr/meanstack_angular:dev .
```

Push the docker image to docker hub with the command:

```
docker push maxeymr/meanstack_angular:dev
```

Verify the new repository is in docker hub.

4. Show Angular application is running
   Change to the meanstack directory and run the command docker-compose up --build to build and run the containers  after a while the three containers with mongo, express, and angular should be available at the following addresses
   mongo: http://localhost:27017/
   express: http://localhost:3000/
   angular: http://localhost:4200/

The angular app will be running at: http://localhost:4200/
You will be able to add Todo items if the mongodb and express containers are running.

Angular app running in a browser with 1 todo item added

Press Ctrl+C to stop the running processes (and the three containers will stop).

## Notes: The following commands are useful with docker compose

```
docker-compose ps will list the containers
docker-compose kill will kill containers
docker-compose down will stop and remove containers and etc.
```

## Advantages and disadvantages of docker and containers

1. Advantages

   - Cost savings in infrastructure.
   - Software standardization.
   - Removes versioning and dependency issues.
   - Improved configurations and building of apps.
   - Good solution for CI/CD of applications.
   - Build once and deploy to multiple environments or operating systems.
   - Applications can run in isolation.

2. Disadvantages
   - Security issues due to common kernel and components of hose OS.
   - Tedious to prepare containers.
   - Requires docker on each machine which runs docker.
   - Data storage concerns where data inside containers is removed when closed (you have to migrate data away from a container before shutting it down).
   - Not designed for all use cases (best suited for microservices implementation.
