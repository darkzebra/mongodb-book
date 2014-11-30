## Indexes

As described under the `$where` query operator, MongoDB has to go through every document in a collection to test if it matches the query operations.  This is called a "collection scan", is very inefficient and will get slower as more records are added to the collection.

Indexes are a way to speed up the querying of collections.  An index creates a data structure that is based upon a field or combination of fields which are stored with a reference to the document.  The fields in the index are ordered by their values.  This greatly speeds up queries when the query uses indexed fields.  It also speeds up sorting on indexed field since they do not need to have an additional sorting step.


### The `_id` Field

All documents are given an `_id` field by default which is an indexed field.  This index cannot be removed.


### Simple Indexes

To create and index on a field, use the `ensureIndex` method of the collection.  This method takes an object as a parameter which defines which field is in the index, i.e. `{field1: 1}`. Creating an index on the year that a book was written would be:

```javascript
db.books.ensureIndex({year: 1})
```

To view the affect of this call, type:

```javascript
db.books.getIndexes()

```

Which will yield the following results:

```javascript
[
	{
		"v" : 1,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "darkzebra.books"
	},
	{
		"v" : 1,
		"key" : {
			"year" : 1
		},
		"name" : "year_1",
		"ns" : "darkzebra.books"
	}
]
```
Now any query using year as a criteria for searching or sorting results is going to perform better.  If you want to drop this index, `db.books.dropIndex({year: 1})` will remove it.  Remember that the parameter document passed to the `dropIndex` must match the one passed to the `ensureIndex`.

#### Unique Indexes

There is a 2nd parameter to the `enureIndex` call which is an object containing additional properties such as `unique`, and `sparse`.  To create a unique index which can only have one matching record (like our `_id` field) we would call `db.books.ensureIndex({isbn: 1}, {unique: 1})`.  This would create a unique index for any `isbn` fields.

#### Embedded Fields and Sub-Documents

Sub-documents and their fields can also be indexed.  The index can be on either a field in a sub-document, called an embedded field, or on the entire sub-document.  Consider the following two examples:

```javascript
db.books.ensureIndex({"author.lastname": 1});
db.books.ensureIndex({author: 1});
```

The first line creates an index on the author's last name, so the `.find({"author.lastname": "Steinbeck"})` call would use this index.  However, the 2nd index is on the entire sub-document and it is affected by the rules about querying sub-documents; the order and names of the fields in the sub-document affect the results.  This also means that it only affects sub-document queries and not embedded fields, so it won't help searches by just the author's last name.

### Compound Indexes

It is not difficult to make indexes with multiples fields, it simply means adding a new field to the parameter in the `ensureIndex` call.

```javascript
db.books.ensureIndex({year: 1, "author.lastname": 1, "author.lastname": 1})
```

#### Index Prefixing

The previous example brings up an important topic in MongoDB Indexing: prefixes.  This means that this index will be used even if not all of the fields are queried on, as long as they are excluded in the reverse order of the index.  It sounds more complicate than it is, so consider the following queries:

```javascript
db.books.find({year: 1936})
db.books.find({year: 1936, "author.lastname": "Mitchell"})
```

Because the indexes are created and sorted by the order that the fields appear in the index document, you can use parts of the index as long as your query still contains the first field.  `year` is the first column in our index, so any query that has the year criteria, can use this index.  However, if we only query on "author.lastname" an exclude the year, then the index will not be used and all of the documents will be scanned.

```javascript
//Will not use the index because it does not include 'year'
db.books.find({"author.lastname": "Mitchell"})
```

### Text Indexes


### Which Indexes Am I Using?

There is an `explain()` method attached the cursor (which is returned by a `find` call) that will give you information on query information, specifically which indexes were used and how many documents had to be searched.
