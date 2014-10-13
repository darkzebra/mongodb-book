#Getting Started with MongoDB
You can simply login by typing `mongo` into the command line and you'll be given a prompt.
```bash
MongoDB shell version: 2.6.4
connecting to: 172.17.0.139/test
>

```
## Database
We can view the databases that are on this server using the following command:

```javascript
show dbs
```
which will output something like this:

```
admin	(empty)
local	0.078125GB
```
To switch between databases type `use <db name>`.  In MongoDB we can use any database (even if it hasn't been created yet).  By simply typing

```javascript
use darkzebra
```
a temporary database has been created.  If we type `show dbs` again, the darkzebra database won't be listed because nothing has been written to it yet.  MongoDB will not actually create the database until we have inserted a record into the database.

## Collections

In MongoDB a collection is roughly equivalent to a table in a relational database in a database like MySQL.  Collections are a series of records that are of the same kinds of data.

One of the key differences between a relational database and a non-relational database like MongoDB is the need for a structured schema when defining the database.  A relational database requires the administrator to define the structure for a table before inserting any record.

Because of this difference we don't need to do define a schema in MongoDB (although you should still have something planned out).  Creating a new collection is the same as creating a new database; just insert a new record, like so:

```javascript
db.quotes.insert({author: 'Shakespeare', quote: 'To be or not to be...'});
```

The `db` object represents the currently selected database and if you type `db` into the command prompt, you'll see the same of the currently selected database.  `db.quotes` represents the collection which will be modified and, like the database, can be created on the fly because it does not need a schema.

> The format of the record is called BSON or Binary Standard Object Notation.  It is very similar to JSON for Javascript but has some additional information about types.  For the most part is is very similar and JSON objects should be valid BSON.

Now that we've inserted a record both the database and the collection have been saved.  Typing `show dbs` will list the following:

```
admin	(empty)
darkzebra	0.078125GB
local	0.078125GB
```

and `show collections`

```
quotes
system.indexes
```
## Schema-less Database
Because MongoDB does not require a schema, it allows us flexibility when we are inserting records into a collection.  We only need to include fields that are set for the record and we can exclude empty fields.  For instance:

```javascript
db.quotes.insert({
  author: {first: "John", middle: "F", last: "Kennedy"},
  quote: 'Ich bin ein Berliner',
  year: 1963
})
```

will insert another record with a `year` field which does not exist on the first record.  It also highlights another feature of MongoDB; sub-documents that are *documents* (or objects) that can be entered as a value for one of the parent document's attributes.  The author attribute for Shakespeare is a string but it is another document for John F. Kennedy.

We can now get all of the documents in our collection by using the `find` method, ie.:

```javascript
> db.quotes.find()
{ "_id" : ObjectId("543069c080ab77c429fb23b8"), "author" : "Shakespeare", "quote" : "To be or not to be..." }
{ "_id" : ObjectId("5430940b610e2dd5c4ca39c9"), "author" : { "first" : "John", "middle" : "F", "last" : "Kennedy" }, "quote" : "Ich bin ein Berliner", "year" : 1963 }
```
We now have two documents in our `db.quotes` collection.  These two objects are similar but have some different fields and a slightly different schema, which is allowed in MongoDB.  Mongo also created an `_id` field automatically which is of the type `ObjectId`.  The [ObjectId](http://docs.mongodb.org/manual/reference/object-id) is MongoDb's unique identifier which is 12 bytes in size and acts as a primary key for the document.  


## Dropping a Collection and Database
Dropping a collections is very simple.

```javascript
db.quotes.drop()
```

and to remove the database:

```javascript
//make sure you have the right one selected
use darkzebra
db.dropDatabase()
```

And now the collection and database are gone.
