##Retrieving and Reading Documents

In the introduction we covered how to see the records in a collection using the `find()` method. Now we will over some more advanced parts of that command to get more granular in our search.

In this section we are going to use a sample database of books.  This data can be downloaded from the Dark Zebra github account at https://github.com/darkzebra/sample-data.  Simply open the file in mongo/books.json, select and copy the entire file, and paste it between the parenthesis of a `db.books.insert()` call.

### Query Criteria

The records that are returned by a `find` call can be confined to a subset that match a certain criteria.  The criteria is passed into the `find` method as the first parameter and is a BSON object called the Query Document.  If an empty Query Document (or no argument) is passed to the method, then all records will be returned.

At their basic level a Query Document is simply a key value pair, where the key is the name of the field to query against and the value is what the key must match.  For instance:

```javascript
db.books.find({title: "War and Peace"})
```
will return all documents that have a title matching "War and Peace".  This is an exact match with case sensitivity, so it won't find documents with a title of "War And Peace" (notice the capital A on the 'and'). MongoDB does support Regular Expressions (RegEx) in the value, so `title: /^war and peace$/i` will perform a case-insensitive search for "war and peace".  This will be covered in a little more detail later.

> Remember that the Query Document is an object, so each field name must be unique.  You cannot put the title field in twice or it will only search for the last value.

What about other operators, other than equals?  Well, these other operators are passed in as a sub-document to the Query Document as a field value.  So, for a number greater than 5, the sub-document would be:

```javascript
{$gt: 5}
```
and the entire query document would then be:

```javascript
{number_field: {$gt: 5}}
```

In the books collection, to find all of the books which have been published since 1900, the call would be:

```javascript
//$gte = greater than or equal to
db.books.find({year: {$gte: 1900}})
```

We can also pass the Query Document into a `count()` method to get the number of documents that match our criteria.

```javascript
db.books.count({year: {$gte: 1900}})
```
All of the criteria that can be used in the Query Document are found in the MongoDB documentation at http://docs.mongodb.org/manual/reference/operator/query/.  The basic operators are:

* $gt, $gte - Greater Than and Greater Than or Equal to.
* $lt, $lt - Less Than and Less Than or Equal to.
* $in, $nin - In and Not In an array, i.e. `{$in: ['War and Peace', 'Mistborn']}`.
* $ne - Not Equal to, the opposite of the default `field: value` search.

#### Logical Operators

By default, adding additional criteria to the Query Document will  make them act as if they were all needed.  In otherwords, they act as a logical *AND*, finding only documents that match all of the criteria.

```javascript
//This will find one record
db.books.find({title: "War and Peace", year: 1897});

//This will not find any records that match
db.books.find({title: "War and Peace", year: 2006});
```

When we get into the logical *OR* in our searches the Query Document becomes a little more complicated.  Their is a criteria operator named `$or` which can accomplish this task.  The `$or` operator is a top level field name which requires a value which is an array.  Let's say we want to find all books published before 1800 and after 2000.

```javascript
db.books.find({
  $or: [ //$or must be an array
    {year: {$lt: 1800}}, //each array element is a sub Query Document
    {year: {$gt: 2000}}
  ]
})
```
The first Query Document contains a `$or` field which has an array value (defined by the brackets).  Each element in the `$or` array must be another Query Document.  It seems complicated at first but you'll get used to it.  If we wanted an *AND* and an *OR* clause in our query then we could do the following:

```javascript
db.books.find({
  title: 'Mistborn',
  $or: [ //$or must be an array
    {year: {$lt: 1800}}, //each array element is a sub Query Document
    {year: {$gt: 2000}}
  ]
})
```
which is the equivalent of saying "find all books with a title of *Mistborn* which have a year before 1800 or after 2000".  In SQL it would be `SELECT * FROM books where title='Mistborn' AND (year<1800 OR year>2000)`.  The other logical operators are:

* `$and` - Finds records that match all criteria.
* `$nor` - Finds all documents that are neither this *NOR* that, will find opposite records of `$or`.
* `$not` - Inverts Query Document to find document that do not match it.

#### Wildcard Searching and Regular Expressions

As shown above, MongoDB support RegEx, which allows us to do some less exact searches on our strings.  The `db.books.find({title: /War/})` call would find both "The Art of War" and "War and Peace".  MongoDB uses Perl Compatible Regular Expressions or PCRE for the Reglar Expressions.  Since RegEx could be a book in itself, I am going to refer you to the Wikipedia page for more details (http://en.wikipedia.org/wiki/Perl_Compatible_Regular_Expressions) and then give a few examples.

* `/^War/` - Must begin with case-sensitive "War".
* `/War/` - Must have the string "War" (case-sensitive) somewhere in the value.
* `/war/i` - Same a above, but case-insensitive.
* `/war.*peace/i` - Matches a string that has "war" followed by "peace" with any number of characters between (case-insensitive).
* `/^war and peace$/i` - Must have an exact match but case-insensitive.

#### Evaluation Operators: $where, $text, $regex, $mod

There are many times where we many need to find documents based upon a more complicated process such as a mathematical model or two properties being equal.  This is where the evaluation operators `$where`, `$text`, `$regex`, and `$mod` come in.

Let's start with `$mod`, or modulus, which is the remainder of a division computation.  If we wanted to only find books that were published on an odd year we could divide the year by 2 and if the remainder is 0, then the year is even, otherwise it is odd.

```javascript
db.books.find({year: {$mod: [2, 1]}})
```
The `$mod` operator takes an array with 2 elements.  The first element is the divisor of the equation, the second element is the remainder value that must match for the record to be retrieved.  So, `{year: {$mod: [2, 1]}}` will return all books whose year, when divided by 2, have a remainder of 1.

The `$regex` was discussed earlier, but allows you to use it in an *AND* query with other operators.  Also, the `$regex` cannot be used inside of an `$in` operator.  The queries `{title: /war/i}` and `{title: {$regex: /war/i}}` are identical except we can add more conditions onto the second.

`$text` will perform a text search on any properties that have a 'text' index type (We will cover indexes laters).  This means it does not specify a specific field, rather it finds all text indexes and searches on those fields.  If there are no fields with that index, you will receive an error.

```javascript
//Unless you have defined an index on books this will throw an error.
db.books.find({$text: {$search: "War"}})
```

Finally, there is the `$where` operator which is very useful.  It is similar to a SQL `WHERE` clause and can perform multiple operations with more complexity.  You should be aware that it has a major drawback; it must be executed on every document in the collection, so it can be very inefficient and slower than other queries.

The `$where` operator can take either a boolean expression string (i.e. `"true !== false"`) or a function that returns a boolean result.  It exposes the `this` object as the current selected object. The following two examples are functionally the same:

```javascript
db.books.find({$where: "this.title.length > 25"})
```
```javascript
db.books.find({$where: function () {return this.title.length > 25}})
```
The `$where` operator is very powerful but has caveats.  It also exposes several variables and functions that you can use in your query.  For more information go to the documentation: http://docs.mongodb.org/manual/reference/operator/query/where/#op._S_where.

#### Null and Missing Properties

In our books collection, there are a few books that do not have years associated with them.  Whenever we do a query using the `year` property, those books are automatically excluded.  This is a nice feature of MongoDB (and NoSQL databases) because it allows for documents that have slightly different schemas.  With that feature, how do we find all of our records that don't have a certain property?  MongoDB provides an `$exists` operator to find these records.

```javascript
db.books.find({year: {$exists: false}})
```
This query will find something like:

```javascript
{ "_id" : ObjectId("5439e9a0fc21c80f8a1a3074"), "title" : "The Art of War", "author" : { "firstname" : "Sun", "lastname" : "Tzu" } }
{ "_id" : ObjectId("5439e9a0fc21c80f8a1a3076"), "title" : "Iliad", "author" : "Homer" }
```
What about searching for `null` (i.e. `db.books.find({year: null})`)?  This will also work but it will also match records that have explicitly set the field value to `null`.  So, `null` will find records without the field and records where it has been set to `null`.  `$exists: false` will only find records where the field has not been set.


#### Querying subdocuments


### Projection (Selecting Fields)

Projections are a way of choosing which fields in a document are returned.  It is an object passed as the 2nd parameter to the `find` method. The object has properties that match the document but they are defined as a 1 or a 0; 1 meaning the field is shown and 0 means the field is not shown.  By default the `_id` field is always returned unless it is explicitly hidden.

```javascript
// Gets the _id and title of all books
db.books.find({}, {title: 1})

//Gets the title of all books (exludes _id)
db.books.find({}, {_id: 0, title: 1})

//Get all books and all properties except year
db.books.find({}, {year: 0})
```



### Limits and Paging


### Cursors
