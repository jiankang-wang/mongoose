Mongoose: MongoDB ODM/ORM
=========================

The goal of Mongoose is to provide a extremely simple interface for MongoDB. 

## Features

- Reduces the burden of dealing with nested callback from async operations.
- Provides a simple yet rich API.
- Ability to model your data with custom interfaces.
- Allows for complex business logic in an async way (see `Promises`).

## How to use

### Setup

In your application, add mongoose to the requires path:

	require.paths.unshift('vendor/mongoose');
	var mongoose = require('mongoose').Mongoose;

### Defining a model

	mongoose.model('User', {
		
		properties: ['first', 'last', 'age', 'updated_at'],
		
		setters: {
			first: function(v){
				return this.v.capitalize();
			}
		},
		
		getters: {
			full_name: function(){ 
				return this.first + ' ' + this.last 
			}
		},
		
		methods: {
			save: function(fn){
				this.updated_at = new Date();
				this.__super__(fn);
			}
		},
		
		static: {
			findOldPeople: function(){
				return this.find({age: { '$gt': 70 }});
			}
		}
		
	});
	
### Getting a model

Models are pseudo-classes that depend on an active connection. To connect:

	var db = mongoose.connect('mongodb://localhost/db');
	
To get a model:
	
	var User = db.model('User');
	
To create a new instance (document)

	var u = new User();
	u.name = 'John';
	u.save(function(){
		sys.puts('Saved!');
	});

## API

### mongoose

#### Methods
	
- **model(name, definition)**
	
	- *definition* determines how the class will be constructed. It's composed of the following keys. All of them are optional:
		
		- *collection*
		
			Optionally, the MongoDB collection name. Defaults to the model name in lowercase and plural. Does not need to be created in advance.
		
		- *properties*
		
			Defines the properties for how you structure the model.
		
			To define simple keys:
		
				properties: [ 'name', 'last' ]
		
			To define arrays:
		
				properties: [ 'name', ['tags'] ]
		
			To define embedded objects:
			
				properties: [ 'name', [{blogposts: ['title', 'body', ...]}] ]
		
			`_id` is added automatically for all models.
		
		- *getters*
		
		- *setters*
		
		- *cast*
		
			Defines type casting. By default, all properties beginning with `_` are cast to `ObjectID`.
		
		- *indexes*
		
		- *methods*
		
		- *static*
		
			The name of the MongoDB collection in your database. If not specified, defaults to the plural of the model name, in lowercase.
	
### Model

These are methods and properties that all *model instances* already include:

#### Properties

- **isNew**
	
	Whether the instance exists as a document or the db (*false*), or hasn't been saved down before (*true*)

#### Methods

Note: if you override any of these by including them in the `methods` object of the model definition, the method is inherited and you can call __super__ to access the parent.

- **save(fn)**
	
	Saves down the document and fires the callback.
	
### Model (static)

These are the methods that can be accessed statically, and affect the collection as a whole.

- **find(props, subset, hydrate)**

	Returns an instance of QueryWriter

	- *props*
	
		Optional, calls the QueryWriter `where` on each key/value. `find({username: 'john'})` is equivalent to:
			
			model.find().where('username', 'john');
		
	- *subset*
	
		Optional, a subset of fields to retrieve. More information on [MongoDB Docs](http://www.mongodb.org/display/DOCS/Advanced+Queries#AdvancedQueries-RetrievingaSubsetofFields)
	
	- *hydrate*
	
		Possible values:
			
			- `true`. Returns a model instance (default)
			- `null`. Returns a plain object that is augmented to match the missing properties defined in the model.
			- `false`. Returns the object as it's retrieved from MongoDB.
	
### QueryWriter

QueryWriter allows you to construct queries with very simple syntax. All its methods return the `QueryWriter` instance, which means they're chainable.

#### Methods

##### Executers

These methods execute the query and return a `QueryPromise`.

- **exec**

Executes the query.

- **count**

Executes the query (and triggers a count)

In addition, for the sake of simplicity, all the promise methods (see "Queueable methods") are mirrored in the QueryWriter and trigger `.exec()`. Then, the following two are equivalent:

	Users.find({username: 'john'})

##### Modifiers
	
- **where**
	
- **sort**

- **limit**

- **skip**

- **snapshot**

- **group**

### QueryPromise

A promise is a special object that acts as a `queue` if MongoDB has not resulted the results, and executes the methods you call on it once the results are available.

For example

	Users.find({ age: { '$gt': 5 } }).first(function(result){
		// gets first result
	}).last(function(result){
		// gets last result
	});

#### Methods

- **stash(fn)**

Stashes all the current queued methods, which will be called when `complete` is called. Methods that are queued **after** stash is called will only fire after `complete` is called again.

- **complete(result)**

Complets the promise. The result parameter is optional. It's either null or an array of documents. (internal use)

##### Queueable Methods

You can call all of these in a row, but the callbacks will only trigger when `complete is called`

- **all(fn)**

Fires with all the results as an array, or an empty array.

- **get(fn)**

Synonym to `all`

- **last(fn)**

Fires with the last document or *null*

- **first(fn)**

Fires with the first document of the resulset or *null* if no documents are returned

- **one(fn)**

Synonym to `first`

## Credits

Nathan White &lt;nathan@learnboost.com&gt;

## License 

(The MIT License)

Copyright (c) 2010 LearnBoost &lt;dev@learnboost.com&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.