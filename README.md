Overview
========

This is a simple, pure-Go, full text indexing and search library.

I made it for use on small to medium websites, although there is nothing web-specific about it's API or operation.

Cdb (http://github.com/jbarham/go-cdb) is used to perform the indexing and lookups.

Usage
------

First, you must create an index.  Like this:

	import "github.com/bradleypeabody/fulltext"

	// create new index with temp dir (usually "" is fine)
	idx, err := fulltext.NewIndexer(""); if err != nil { panic(err) }
	defer idx.Close()

	// for each document you want to add, you do something like this:
	doc := fulltext.IndexDoc{
		Id: []byte(uuid), // unique identifier (the path to a webpage works...)
		StoreValue: []byte(title), // bytes you want to be able to retrieve from search results
		IndexValue: []byte(data), // bytes you want to be split into words and indexed
	}
	idx.AddDoc(doc) // add it

	// when done, write out to final index
	err = idx.FinalizeAndWrite(f); if err != nil { panic(err) }

Once you have an index file, you can search it like this:

	s, err := fulltext.NewSearcher("/path/to/index/file"); if err != nil { panic(err) }
	defer s.Close()
	sr, err := s.SimpleSearch("Horatio", 20); if err != nil { panic(err) }
	for k, v := range sr.Items {
		fmt.Printf("----------- #:%d\n", k)
		fmt.Printf("Id: %s\n", v.Id)
		fmt.Printf("Score: %d\n", v.Score)
		fmt.Printf("StoreValue: %s\n", v.StoreValue)
	}

It's rather simplistic.  But it's fast and it works.

TODOs
-----

* Will likely need some sort of "stop word" functionality.

* Wordize(), Indexize() and the scoring aggregation logic should be extracted to callback functions with the existing functionality as default.

* If there is some decent b-tree disk storage that is portable then it would be worth looking at using that instead of CDB and implementing LIKE-style matching.  As it is, CDB is quite efficient, but it is a hash index.


Implementation Notes
--------------------

I originally tried doing this on top of Sqlite.  It was dreadfully slow.  Cdb is orders of magnitude faster.

Two main disadvantages from going the Cdb route are that the index cannot be edited once it is built (you have to recreate it in full), and since it's hash-based it will not support any sort of fuzzy matching unless those variations are included in the index (which they are not, in the current implementation.)   For my purposes these two disadvantages are overshadowed by the fact that it's blinding fast, easy to use, portable (pure-Go), and it's interface allowed me to build the indexes I needed into a single file.

In the test suite is included a copy of the complete works of William Shakespeare (thanks to Jeremy Hylton's http://shakespeare.mit.edu/) and this library is used to create a simple search engine on top of that corpus.  By default it only runs for 10 seconds, but you can run it for longer by doing something like:

	SEARCHER_WEB_TIMEOUT_SECONDS=120 go test fulltext -v
