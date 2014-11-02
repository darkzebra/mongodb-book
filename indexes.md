## Indexes

As described under the `$where` query operator, MongoDB has to go through every document in a collection to test if it matches the query operations.  This is called a "collection scan", is very inefficient and will get slower as more records are added to the collection.

Indexes are a way to speed up the querying of collections.  An index creates a data structure that is based upon a field or combination of fields which are stored with a reference to the document.  The fields in the index are ordered by their values.  This greatly speeds up queries.


### The `_id` Field

All documents are given an `_id` field by default which is an indexed field.  This index cannot be removed.
