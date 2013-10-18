fulltext
========

This is a simple, pure-Go, full text indexing and search library.

I made it for use on small to medium websites, although there is nothing web-specific about it's API or operation.

In the test suite is included a copy of the complete works of William Shakespeare and a this library is used to create a simple search engine using that data.

USAGE:
------

First, you must create an index.  Like this:

	// create new index with temp dir (usually "" is fine)
	idx, err := NewIndexer(""); if err != nil { panic(err) }

	

TODO:
-----

* We'll likely need some sort of "stop word" functionality.

* Wordize(), Indexize() and the scoring aggregation logic should be extracted to callback functions with the existing functionality as default.

* If there is some decent b-tree disk storage that is portable then it would be worth looking at using that instead of CDB and implementing LIKE-style matching.  As it is, CDB is quite efficient, but it is a hash index.


IMPLEMENTATION NOTES:
---------------------

I originally tried doing this on top of Sqlite.  It was dreadfully slow.  Cdb is orders of magnitude faster.  Two main disadvantages from going the Cdb route are that the index cannot be edited once it is built (you have to recreate it in full), and since it's hash-based it will not support any sort of fuzzy matching unless those variations are included in the index (which they are not, in the current implementation.)   For my purposes these two disadvantages are overshadowed by the fact that it's blinding fast, easy to use, portable (pure-Go), and it's interface allowed me to build the indexes I needed into a single file.
