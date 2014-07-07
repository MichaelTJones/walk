walk
====

Fast parallel version of golang filepath.Walk()

Performs traversals in parallel so set GOMAXPROCS appropriately. Vaues of 8 to 16 seem to work best on my 
4-CPU plus 4 SMT pseudo-CPU MacBookPro. The result is about 4x-6x the traversal rate of the standard Walk().
The two are not identical since we are walking the file ystem in a tumult of asynchronous walkFunccalls any
number of goroutines. So, take note of the following:

1. This walk honors all of the walkFunc error semantics but as multiuple user-supplied 
walkFuncs may simultaneously encounter a traversal error or generate one to stop traversal, only the FIRST
of these will be returned as the Walk() result. Further, since there may be a few files in flight at the 
moment of the error discovery, a few more walkFunc calls may happen after the first error-generating call
has signaled its desire to stop. In general this is a non-issue but it could be so pay attention when 
designing your walkFunc.

2. Because the walkFunc is called concurrently in multiple goroutines, it needs to be careful about what it does with external data. Results may be printed using fmt, but generally the best plan is to send results over a channel or accumulate counts using a locked mutex.

Both of these issues are illustrated/handled in the simple traversal programs supplied with walk.
