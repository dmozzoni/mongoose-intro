# Intro to Mongoose

## Learning Objectives

- Differentiate between NoSQL and SQL databases
- Explain what Mongoose is
- Describe the role of Mongoose schema and models
- Use Mongoose to perform CRUD functionality
- List and describe common Mongoose queries
- Persist data using Mongoose embedded documents

## Opening Framing

In previous WDI units, we used ActiveRecord to interact with and perform CRUD actions on a SQL database through a Ruby back-end. Today, we'll be doing the equivalent with a tool called Mongoose on a NoSQL database using a Node back-end.

Before we dive into Mongoose, however, let's talk a bit about about Mongo and NoSQL databases. **What are some of your takeaways?**

<details>
  <summary><strong>What is a NoSQL database?</strong></summary>

  A NoSQL database is a non-relational database.
  * This means no explicit one-to-one, one-to-many and many-to-many relationships.
  * That being said, we can emulate these relationships in a NoSQL database.

</details>

<details>
  <summary><strong>How is a NoSQL database organized?</strong></summary>

  A NoSQL database can be organized into **documents** and **collections**.
  * Collections are the NoSQL equivalent of tables in a SQL database.
  * Documents are the NoSQL equivalent of a table row.

  > This is not the only way that a NoSQL database organizes data. Start [here](http://rebelic.nl/2011/05/28/the-four-categories-of-nosql-databases/) to learn more.

</details>

<details>
  <summary><strong>What is MongoDB?</strong></summary>

  A NoSQL database that stores information as JSON.
  * Technically, it's BJSON -- "binary JSON."
  * Think of this as taking the place of Postgres.

</details>

#### Why Use NoSQL/Mongo Over SQL?

It's flexible.
* You don't need to follow a schema if you don't want to. This might be helpful with non-uniform data.
* That being said, you can enforce consistency using schemas. In fact, we'll be doing that in today's class.

It's fast.
* Data is "denormalized" in a NoSQL data, meaning that it's all in the same place.
* For example, a post's comments will be nested directly within the post in the database.
* This is unlike a relational database, in which we need to make queries to retrieve data connected through a relation.

Many web apps already implement object-oriented Javascript.
* If we're using objects in both the back-end and front-end, that makes handling and sending data between the client and a database much easier.
* No need for type conversion (e.g. making sure a Ruby hash is being served as JSON).

#### Example MongoDB Commands

Even though you won't be writing much Mongo in WDI, we will be using some MongoDB CLI commands to test what's in our database in this class. Examples include...
* `show dbs` - Show a list of all databases
* `use database-name` - Connect to a database
* `show collections` - List the collections in a database
* `db.authors.find()` - List all authors in an author collection

## Mongoose

![mongoose.js](https://www.filepicker.io/api/file/KDQZV88GTIaQn6p0GagE)

> "Let's face it, writing MongoDB validation, casting and business logic boilerplate is a drag. That's why we wrote Mongoose."

Mongoose is an ODM (Object Data Mapping), that allows us to encapsulate and model our data in our applications. It gives us access to additional helpers, functions and queries to simply and easily preform CRUD actions.

Mongoose will provide us with the similar functionality to interact with MongoDB and Express as Active Record did with PostgreSQL and Rails.

> Active Record is an ORM (Object Relational Mapping). An ODM is the same thing without relations.

Here's an example of some Mongoose code pulled from  [their documentation](http://mongoosejs.com).

```js
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/test');

var Cat = mongoose.model('Cat', { name: String });

var kitty = new Cat({ name: 'Zilda' });
kitty.save(err => {
  if (err) // ...
  console.log('meow');
});
```

## You Do: Initial Set Up

During today's in-class exercises, you will be creating a two-model todo app using Mongoose and MongoDB.

It will have two models: Authors and Reminders.
* Each Author has a `name` (string) and an `age` (number)
* Each Reminder has a `body` (string)

Authors will have many Reminders, although we won't be implementing that using a SQL relationship.

Follow these steps...
  1. Fork and Clone [this repo](https://github.com/ga-wdi-exercises/reminders_mongo): `$ git clone git@github.com:ga-wdi-exercises/reminders_mongo.git`
  2. Make sure to checkout locally to the `mongoose` branch: `$ git checkout mongoose`
  3. Also make sure you are running `$ mongod` in a separate Terminal tab.

> [Starter Code](https://github.com/ga-wdi-exercises/reminders_mongo/tree/mongoose)
>
> [Solution Code](https://github.com/ga-wdi-exercises/reminders_mongo/tree/mongoose-solution)

## I Do: Mongoose and Connection Set Up

After each demo, you will apply the same functionality to your Reminders app.

> This means you should not be following along when I am building the Reminders app.

Let's begin by installing Mongoose...

```bash
$ npm install --save mongoose
```

In order to have access to `mongoose` in our application, we need to explicitly require mongoose and open a connection to test the database on locally...

```js
// db/schema.js

var mongoose = require("mongoose");
mongoose.connect("mongodb://localhost/authors");
```

> The name above `authors` will be the name of the database stored in MongoDB

```js
// db/schema.js
var mongoose = require("mongoose");
mongoose.connect("mongodb://localhost/authors");

// Now that we're connected, let's save that connection to the database in a variable.
var db = mongoose.connection;

// Will log an error if db can't connect to MongoDB
db.on("error", err => {
    console.log(err);
});

// Will log "database has been connected" if it successfully connects.
db.once("open", () => {
    console.log("database has been connected!");
});
```

Now let's run our `db/schema.js` file...

```bash
$ node db/schema.js
```

## You Do: Install Mongoose and Connection

Follow the instructions in the previous section to require mongoose in your Reminders app.

> [Solution](https://github.com/ga-wdi-exercises/reminders_mongo/blob/267a908faaae06ab4c35da6d671a867cf1bc6426/db/schema.js)

## I Do: Mongoose Schema & Models

#### What Is a Mongoose Schema?

* Schemas are used to define attributes and structure for our documents.
* Each Schema maps to a MongoDB collection and defines the shape of the documents within that collection.

Here's an example of a Mongoose schema...

```js
// db/schema.js

// First, we instantiate a namespace for our Schema constructor defined by mongoose.
var Schema = mongoose.Schema;

var AuthorSchema = new Schema({
  name: String,
  age: Number
});

var ReminderSchema = new Schema({
  body: String
});
```

#### What Are Mongoose Models?

Mongoose Models will represent documents in our database.
* Models are defined by passing a Schema instance to `mongoose.model`

```js
// db/schema.js

var Schema = mongoose.Schema;

var AuthorSchema = new Schema({
  name: String,
  age: Number
});

var ReminderSchema = new Schema({
  body: String
});

var Author = mongoose.model("Author", AuthorSchema);
var Reminder = mongoose.model("Reminder", ReminderSchema);
```

`.model()` makes a copy of a schema.
* The first argument is the singular name of the collection your model is for. Mongoose automatically looks for the plural version of your model name when creating a collection.
* That means the `Author` model is for the `authors` collection in the database.

## Collections: Embedded Documents & References

Let's add another model to `db/schema.js`.
* We will be adding a schema for `Reminder` since we want to create an application that tracks Authors and Reminders
* Like a one-to-many relationship in a relational database, an Author will have many Reminders.

With ActiveRecord, we defined a one-to-many relationship like so...

```rb
class Author < ApplicationRecord
  has_many :reminders
end


class Reminder < ApplicationRecord
  belongs_to :author
end
```

In Mongoose, we will do this using **embedded documents**.

## I Do: Embedded Documents

[Embedded Documents](http://mongoosejs.com/docs/2.7.x/docs/embedded-documents.html) -- sometimes referred to as "sub-documents" -- are schemas of their own which are elements of a parent document's array
* They contain all the same features as normal documents.


```js
// db/schema.js

var Schema = mongoose.Schema;

var ReminderSchema = new Schema({
  body: String
});

var AuthorSchema = new Schema({
  name: String,
  age: Number,
  reminders: [ReminderSchema]
});

var Author = mongoose.model("Author", AuthorSchema);
var Reminder = mongoose.model("Reminder", ReminderSchema);
```
> The reminders key of your `AuthorSchema` documents will contain a special array that has specific methods to work with embedded documents.
>
> The Reminder Schema must be defined prior to our main Author Schema.

#### Advantages

* Easy to conceptualize and set up.
* Can be accessed quickly.

#### Disadvantages

* Don't scale well. Documents cannot exceed 16MB in size.

> If you find that you are nesting documents within documents for 3+ levels, you should probably look into a relational database.

## I Do: Multiple Collections & References

Similar to how we use foreign keys to represent a one-to-many relationship in Postgres, we can add [references](https://docs.mongodb.org/manual/tutorial/model-referenced-one-to-many-relationships-between-documents) to documents in other collections by storing an array of `ObjectIds` referencing document ids from another model.

```js
// db/schema.js

var Schema = mongoose.Schema;
var ObjectId = Schema.ObjectId;

var Schema = mongoose.Schema;

var ReminderSchema = new Schema({
  body: String
});

var AuthorSchema = new Schema({
  name: String,
  age: Number,
  reminders: [{type: Schema.ObjectId, ref: "Reminder"}]
});

var Author = mongoose.model("Author", AuthorSchema);
var Reminder = mongoose.model("Reminder", ReminderSchema);
```

> Since we are using an id to refer to other objects, we use the ObjectId type in the schema definition. The `ref` attribute must match the model used in the definition.

#### Advantages

* Could offer greater flexibility with querying.
* Might be a better decision for scaling.

#### Disadvantages

* Requires more work. Need to find both documents that have the references (i.e., multiple queries).

## You Do: Set Up Schema and Models

Use the previous section to step up your Reminder and Author schemas and models.

> [Solution](https://github.com/ga-wdi-exercises/reminders_mongo/blob/cfee42d3cfd0bf5f2581cc61ba712eb8e1b7777f/db/schema.js)

## Break

## I Do: Add Create

First let's create an instance of our Author model. Here's one way of doing it...

```js
// db/schema.js

// First we create a new author. It's just like generating a new instance with a constructor function!
var sora = new Author({name: "Sora", age: 16});

// Then we save it to the database using .save
sora.save((err, author) => {
  if(err){
    console.log(err);
  }
  else{
    console.log(author);
  }
});
```

> You can compare this to `.new` and `.save` in ActiveRecord.

We can also consolidate that into a single `.create` method, like so...

```js
// db/schema.js

Author.create({ name: "Sora", age: 16 }, (err, author) => {
  if (err){
    console.log(err);
  }
  else{
    console.log(author);
  }
});
```

## Callback Functions

Oftentimes, when making a Mongoose query we will pass in a callback function. It will be passed two arguments: `err` and `data`.
* `err` will contain an error message if something goes wrong with the Mongoose query
* `data` contains the result of the Mongoose query

<details>

  <summary><strong>Why do you think callbacks might be necessary when using Mongoose?</strong></summary>

  > Because these queries are asynchronous! We want to make sure the query has finished before we run any code that depends on the result.

</details>

## Promises

If callbacks aren't your cup of tea, you can replace the callbacks we used above with promise methods.

```js
var sora = new Author({name: "Sora", age: 16});

sora.save().then(author => {   // We don't pass in `err` as an argument here...
  console.log(author)
}).catch(err => {               // ...instead, we pass it in here
  console.log(err)              // If there's an error, this `.catch` method will be triggered
});
```

## I Do: Add Embedded Documents

Next, let's create a Reminder...

```js
// db/schema.js
var sora = new Author({name: "Sora", age: 16});
var newReminder = new Reminder({body: "Don't forget to set up your schema properly"});

// Now we add that reminder to an author's collection / array of reminders.
sora.reminders.push(newReminder)

// In order to save that reminder to the author, we need to call `.save` on the author -- not the reminder.
sora.save((err, author) => {
  if(err){
    console.log(err)
  } else {
    console.log(`${author} was saved to our db!`);
  }
});
```
## You Do: Add Create
Make sure the above sections are added to your code.
> [Solution](https://github.com/ga-wdi-exercises/reminders_mongo/commit/9b5a93841df550516e04778066cb43bd790c11f8)

## I Do: Seed Data

Let's seed some data in our database. In order to do that, we need to first make sure we can connect `schema.js` to `seeds.js`. Let's add the following to `db/schema.js`...

```js
// db/schema.js

// The rest of our schema code is up here...

// By adding `module.exports`, we can now reference these models in other files by requiring `schema.js`.
module.exports = {
  Author: Author,
  Reminder: Reminder
};
```

And add the following to `db/seeds.js`...

```js
// db/seeds.js

var Schema = require("./schema.js");

var Author = Schema.Author;
var Reminder = Schema.Reminder;
```

Now let's call some methods in `db/schema.js` that will populate our database...

```js
// db/seeds.js

var Schema = require("./schema.js");

var Author = Schema.Author;
var Reminder = Schema.Reminder;

// First we clear the database of existing authors and reminders.
Author.remove({}, err => {
  if(err){
    console.log(err)
  }
});

Reminder.remove({}, err => {
  if(err){
    console.log(err)
  }
});

// Now we generate instances of Author and Reminder.
var george = new Author({name: "George", age: 68});
var joanne = new Author({name: "Joanne", age: 51});
var tom = new Author({name: "Tom", age: 71});

var reminder1 = new Reminder({body: "Reminder 1: Learn JS"});
var reminder2 = new Reminder({body: "Reminder 2: Learn Rails"});
var reminder3 = new Reminder({body: "Reminder 3: Learn Angular"});
var reminder4 = new Reminder({body: "Reminder 4: Learn Express"});

var authors = [george, joanne, tom];
var reminders = [reminder1, reminder2, reminder3, reminder4];

// Here we assign some reminders to each author.
for(var i = 0; i < authors.length; i++){
  authors[i].reminders.push(reminders[i], reminders[i+1])
  authors[i].save((err, author) => {
    if (err){
      console.log(err)
    } else {
      console.log(author);
    }
  })
};

// ...or you could use forEach instead of a for loop...
authors.forEach((author, i) => {
  author.reminders.push(reminders[i], reminders[i+1])   // Assigning each author multiple reminders
  author.save((err, author) => {
    if (err){
      console.log(err)
    } else {
      console.log(author);
    }
  })
})

```
Now, seed your database by running `node db/seeds.js` in your terminal. Use Ctrl + C to exit running Node.

Let's test if this all worked by opening Mongo in the Terminal...

```bash
$ mongo
$ show dbs
$ use authors
$ show collections
$ db.authors.find()
```

## You Do: Add Seed Data

Now do the same thing with your Reminders app.

> [Solution](https://github.com/ga-wdi-exercises/reminders_mongo/commit/9b5a93841df550516e04778066cb43bd790c11f8)

## I Do: Mongoose Queries

Like Active Record, Mongoose provides us with a variety of helper methods that allow us to easily retrieve documents from our database.

> Explore them using the [Mongoose Queries Documentation](http://mongoosejs.com/docs/api.html#query-js).

```js
// Finds all documents of a specified model type. We can pass in a key-value pair(s) to narrow down the search.
Model.find({}, callback)

// Finds a single model by its id.
Model.findById(someId, callback)

// Find a single model using a key-value pair(s).
Model.findOne({someKey: someValue}, callback)

// Removes documents that match a key-value pair(s).
Model.remove({someKey: someValue}, callback)
```

Let's use `.find` to implement `index` functionality. We'll do that in a controller file...

```bash
$ mkdir controllers
$ touch controllers/authorsController.js
```

> We are adding a `controllers` directory and `authorsController.js` file to mimic how we might define a controller in an Express application. Like how our controllers helped us in Rails, we will be following similar REST conventions and using our controllers to listen for incoming requests and communication with our database.


```js
// controllers/authorsController.js

var Schema = require("../db/schema.js");
var Author = Schema.Author;
var Reminder = Schema.Reminder;

var authorsController = {
  index(){
    Author.find({}, (err, authors) => {
      console.log(authors);
    });
  }
};

authorsController.index();
```

Run `$ node controllers/authorsController.js` in the terminal.

Now let's do `show`...

```js
// controllers/authorsController.js

var authorsController = {
  index(){
    Author.find({}, (err, authors) => {
      console.log(authors);
    });
  },
  show(req){
    Author.findOne({name: req.name}, (err, author) => {
      console.log(author);
    });
  }
};

authorsController.show({name: "Tom"});
```

> `req` here stands for "request." While we don't have to name the argument that way, it is common practice. It represents information that is being sent to a server. In this case, it contains information about the data in question: a single author.

## You Do: Index, Show, Update and Delete

Follow the above instructions to implement `index` and `show` for the Author model.

Then use [Mongoose documentation](http://mongoosejs.com/docs/api.html#query-js) to figure out how to update and delete authors. If the documentation proves difficult to navigate, don't be afraid to Google it! We'll go over how to update and delete after the exercise...

> [Index/Read Solution](https://github.com/ga-wdi-exercises/reminders_mongo/commit/d51081c0bf995bbd7f47883467da1c06a03de058)
> [Update Solution](https://github.com/ga-wdi-exercises/reminders_mongo/commit/d51081c0bf995bbd7f47883467da1c06a03de058)
> [Delete Solution](https://github.com/ga-wdi-exercises/reminders_mongo/blob/469d3c09059c60b7779a8c3a8c2fb12aefcc779a/controllers/authors.controller.js)

#### Bonus

If you finish early...
* Create a controller method that returns the number of Authors or Reminders in the database
* Create a controller method that clears all of an Author's Reminders (i.e., the author has no reminders)
* Check out the Validations section at the end of this lesson

## Break

## I Do: Update & Delete

<details>
  <summary><strong>This is how to <code>update</code>...</strong></summary>

  ```js
  // controllers/authorsController.js
  var authorsController = {

    // This method takes two arguments: (1) the old instance and (2) what we want to update it with.
    update(req, update){
      Author.findOneAndUpdate({name: req.name}, {name: update.name}, {new: true}, (err, author) => {
        if(err) {
          console.log(err)
        }
        else {
          console.log(author);
        }
      });
    }
  };

  authorsController.update({name: "Tom"}, {name: "Voldemort"});
  ```

  > We are inserting {new: true} as an additional option. If we do not, we will get the old document as a return value -- not the updated one.

</details>

<details>

  <summary><strong>This is how to <code>delete</code>...</strong></summary>

  ```js
  // controllers/authorsController.js

  var authorsController = {
    destroy(req){
      Author.findOneAndRemove(req, (err, docs) => {
        if(err){
          console.log(err);
        }
        else{
          console.log(docs);
        }
      });
    }
  };

  authorsController.destroy({name: "George"});
  ```

</details>

> Don't look at these while working on the previous exercise!

-----

## Bonus: Validations

Mongoose contains built-in validators and an option to create custom validators as well.

Validators are defined at the field level of a document and are executed when the document is being saved. If a validation error occurs, the save operation is aborted and the error is passed to the callback.

**Built in Validators:**

* `required` and `unique`: used to validate the field existence in Mongoose, which is placed in your schema on the field you want to validate.

Example: Let's say you want to verify the existence of a username field and ensure it's unique before you save the user document.

```js
var UserSchema = new Schema({
  ...
  username: {
    type: String,
    unique: true,
    required: true
  }
});

```
>This will validate the existence and uniqueness of the username field when saving the document, thus preventing the saving of any document that doesn't contain that field.

* `match`: type based validator for strings, placed in your fields for you schemas

Continuing off the above example, to validate your email field, you would need to change your UserSchema as follows:

```js
var UserSchema = new Schema({
  username: {
    type: String,
    unique: true,
    required: true
  },
  email: {
    type: String,
    match: /.+\@.+\..+/
  }
});
```

>The usage of a match validator here will make sure the email field value matches the given regex expression, thus preventing the saving of any document where the e-mail doesn't conform to a valid pattern!

* `enum`: helps to define a set of strings that are only available for that field value.

```js
var UserSchema = new Schema({
  username: {
    type: String,
    unique: true,
    required: true
  },
  email: {
    type: String,
    match: /.+\@.+\..+/
  },
  role: {
    type: String,
    enum: ['Admin', 'Owner', 'User']
  },

});
```
>By Adding in `enum`, we are adding a validation to ensure only these three possible strings are saved in the document.

### Custom Validations

We can also define our own validators by using the `validate property`.

This `validate property` value is typically an array consisting of a validation function.

For example, if we want to validate the length of your user's password. To do so, you would have to make these changes in your UserSchema:

```js
var UserSchema = new Schema({
  ...
  password: {
    type: String,
    validate: [
      function(password) {
        return password.length >= 6;
      },
      'Password should be longer'
    ]
  },
});
```
>This custom validator will make sure your user's password is at least six characters long or it will prevent it from saving the document

* `.pre` validation: middleware that are functions which are passed control during execution of asynchronous functions.

By using `.pre`, these are executed before validations.

```js
UserSchema.pre("save", function(next) {
    var self = this;

    UserModel.findOne({email : this.email}, 'email', function(err, results) {
        if(err) {
            next(err);
        } else if(results) {
            console.warn('results', results);
            self.invalidate("email", "email must be unique");
            next(new Error("email must be unique"));
        } else {
            next();
        }
    });
});

```

-----

## Review Questions

* How is Mongoose used to interact with MongoDB?
* What are embedded documents in Mongoose?
* Why do we create a Schema in Mongoose?
* What are 3 Mongoose query methods?
* **Bonus**: What are common built in validations for Mongoose? Why would we use them?

## Homework

After this class you should be able to complete Part I of [YUM](https://github.com/ga-wdi-pvd/yum).

## Additional Resources

* [GA DC Lesson](https://github.com/ga-wdi-lessons/mongoose-intro)
* [Mongoose Documentation](http://mongoosejs.com/index.html)
* [Embedded Docs versus Multiple Collections](https://www.google.com/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=mongoose%20embedded%20versus%20collections)
* [Active Record Versus Mongoose](active_record_comparison.md)
* [ODM For Mongo and Rails](https://github.com/mongodb/mongoid)
