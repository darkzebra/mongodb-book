## Creating Documents

New MongoDB records are created using the `.insert()` method of our collection.  It does not matter if the collection exists because it will be created once `insert` has been called.  The `insert` method accepts a BSON string that represents the document as a parameter.

```javascript
db.books.insert({
  title: 'War and Peace',
  author: 'Leo Tolstoy',
  year: 1869
});
```
This code will create the `books` collection and insert a new document.  The `insert` method returns a `WriteResult` object with a property of `nInserted` which specifies how many documents were inserted into the collection.  When we execute a `db.books.find()` there will now be a record like:

```javascript
{ "_id" : ObjectId("5432979c9a3f1d26a8e9f5fd"), "title" : "War and Peace", "author" : "Leo Tolstoy", "year" : 1869 }
```
As explained in the overview, an `_id` attribute is automatically created for the document unless one is specified in the insert.  Consequently, the code:

```javascript
db.books.insert({
  _id: 1,
  title: 'The Hobbit',
  author: 'J.R.R Tolkien',
  year: 1937
});
```
will give us a a new record with and `_id` of 1.

```javascript
{ "_id" : 1, "title" : "The Hobbit", "author" : "J.R.R Tolkien", "year" : 1937 }
```

### Insert Multiple Records

MongoDB also allows the inserting of multiple records at the same time.  This is done use multiple BSON objects inside of a BSON array, specified  by brackets `[]`.

```javascript
db.books.insert([
  {title: 'Alice in Wonderland', author: 'Lewis Carroll', year: 1865},
  {title: 'Mistborn', author: 'Brandon Sanderson', year: 2006}
]);
```
Inserting multiple records is called a bulk insert and the `insert` method will now return a `BulkWriteResult` object with a `nInserted` value of 2.
