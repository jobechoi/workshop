# Setting up API Routes and Angular Routes
This part of the workshop should take no more than 75 minutes.

We will create routes on the server to let the server know what we need to do whenever we receive an HTTP request for a particular URL. A common convention for defining private routes for use by your application is to prefix the routh with `/api/`.

We're going to be using the Express framework as a way to simplify our server code. Request handler functions built with Express will specify a particular HTTP verb and a particular route. The request handlers take a URL path and a function as arguments. 

The function passed to the request handler takes two arguments: the request object (sent through the HTTP request) and the response object (what the server sends back to the requesting agent). From the request, we can get information about the user, the data the user has input into the front-end, and many other things. The response is an object that we manipulate in the request handler and send back to the requesting agent.

## Creating a RESTful API
Here are the routes we will build on the server:

|HTTP Verb| URL | Action  |
|---|---|---|
| GET  | /api/todos   | retrieve all of the items from the database  |
| POST |  /api/todos |  add a new item to the database |
| DELETE | /api/todos/:todo_id   |  delete a single item from the database |

Inside each route, we'll use methods provided to us by Mongoose in order to retrieve and manipulate data within the Mongo Database.

###Define `/GET` route
Open `server.js` as we're going to define our routes there.

We'll first create a request handler for the `/GET` route for `/api/todos/`. Here we're using the [Mongoose Find](http://mongoosejs.com/docs/queries.html) function to query the database:

```javascript
// Set up the `/GET` route for `/api/todos`
app.get('/api/todos', function(request, response){
	// Use Mongoose's .find() method to retrieve all todos from database
   	ToDo.find(function(err, todos){
      	// Error handling - if we receive an error from Mongo, send it back as a response. Nothing after res.send(err) will execute
       	if (err) {
         	response.send(err);
        }
       	// If no error, respond with the todos as a json object
        response.json(todos);
   	});
});
```

Take a minute to "talk" through this bit of code. Imagine you had to explain what it does to a five year old child. What do you think this line of code does `response.json(todos);`? 

###Define `/POST` route
Next, we create a request handler for the `/POST` route for `/api/todos/`. Here we're using the [Mongoose Create](http://mongoosejs.com/docs/models.html) function to create a new document and add it to the database.

```javascript
// Set up the `/POST` route for `/api/todos`
app.post('/api/todos', function(request, response){
  	// Use Mongoose's .create() method to create a new item. 
  	ToDo.create({
      	// The text comes from the AJAX request sent from Angular.
       	text: request.body.text,
        complete: false
   		}, function(err, todo){
        	// Error Handling
           	if (err) { 
             	response.send(err); 
        	}
         	// Retrieve all Todos after creating another, in order to repopulate the entire list on the page
            ToDo.find(function(err, todos){
               	if (err) {
                  	response.send(err);
                }
               	response.json(todos);
          	});
    });
});
 ```
 
Did you notice the two parameters `request` and `response`? Why do you think they're important?

###Define `/DELETE` route
The last request handler we create is for the `/DELETE` route for `/api/todos/:todo_id`

```javascript
app.delete('/api/todos/:todo_id', function(request, response){
	ToDo.remove({
   		_id: request.params.todo_id
    }, function(err, todo){
      	if (err) {
        	response.send(err);
      	}
      	// Retrieve all Todos after deleting one, in order to repopulate the entire list on the page
      	ToDo.find(function(err, todos){
        	if (err) {
          		response.send(err);
        	}
        	response.json(todos);
      	});
    });
});
```
###Define public routes (Serving up static files)

Next, we need to set up the route that will be public facing - the route that will return the static HTML file to the client. This HTML file will render the ToDo view.

|HTTP Verb| URL | Action  |
|---|---|---|
| GET  | * | render the static file index.html  |

Create a request handler for a `GET` request to `*`, to handle all `GET` requests to otherwise unspecified routes: 

```javascript
app.get('*', function(request, response){
    response.sendFile(__dirname + '/public/index.html');
});
```

## Next Section

[Setting up the front-end](./branch2.md)
