
`re-frame-10x` captures trace data and, sometimes when debugging, 
you'd like to experiment with this data in your REPL. This document 
explains how to make that happen.

## The Concepts

Just to be clear, we're talking here about the kind of 
REPL offered by figwheel - one that is connected to a browser running your app.

At such a REPL, if you were to
type in `(my-ns/some-fn  "blah" 2)` and hit return, 
then:
 1. this code will compiled within the context of the REPL's "current namespace" 
 2. the resulting (javascript) code will be shipped across to the browser 
 3. that javascript code will be run/evaulated within the context of your running app 

**The Step 1** compilation can use any part of your app's codebase, so long as it 
has been previously `required`. The example code fragment above accessed `some-fn` 
in the namespace `my-ns`, so to make it work, something like the following would
previously have happened in your REPL: 
```clj
(require '[my-ns])
```
or maybe
```clj
(require '[some-app.mine :as my-ns])
```

**For Step 3**, when the code is executing in the browser, it can access any 
part of your running app. 

## The Running App

Until this moment, it may not have occured to you that `re-frame-10x` is 
a part of your app. Yes, it may look seperate, but it is running in the same browser VM 
as your app, and it is a code dependency of your app, and so it can 
be accessed from the REPL like any other part of your app.

So, if we knew enough about the way `re-frame-10x` stores
trace data, we could use the REPL to reach into `re-frame-10x's` 
data structures and access any trace data we wanted.

Imagine that `re-frame-10x` had an API function called, say, `get-trace` 
in a namespace called, say, `tenX`, then you could evaluate the following at the REPL: 
``` 
(tenX/get-trace [:some :identifing :path :to :the :trace 1234])
```
and this call would return the trace data at the given "path".  

Of course, previously you would have `required` this `tenX` namespace so you had access to the API.

WARNING: there's no `get-trace` or `tenX` - they are just my teching aid
to explain the concept, so don't try to type them in at a REPL. 

## The Method 

To facilitate REPL use, `re-frame-10x` writes (ClojureScript) code into the clipboard.
You then paste this code into your REPL to obtain access to trace data.

So, initially, you click on the "XXXX" hyperlink, and `re-frame-10x` will 
put into the clipboard the `require` code needed to access the `re-frame-10x` API.  
You then paste that code into your REPL and execute it. 
 
Then, later, in the `re-frame-10x` UI you'll notice a small "repl" 
button and, again, if you click it, `re-frame-10x` will put code into 
the clipboard which uses its own API and a "path" to 
to access that specific bit of trace data.  

When you paste this code into the REPL, you get access to the exact 
piece of trace data you want.

So, you might end up executing somethink this at the REPL: 
```clj
(count (tenX/get-trace [:some :identifing :path]))
```

That `(tenX/get-trace [:some :identifing :path])` part is what you pasted in
from the clipboard. 

Or maybe you'd do this
```clj
(let [trace-data  (tenX/get-trace [:some :identifing :path])]
    (count trace-data))
```
Or:
```cljs
(def tmp  (tenX/get-trace [:some :identifing :path]))
(count tmp)
```

<!-- put screenshots/gif in here -->


## Why This Way?

This approach carefully avoids the often-problematic need to 
serialise trace data held by `re-frame-10x` in the browser and then 
transfer it over the wire to the REPL for use. 

**First,** when you are tracing large amounts of data, serialisation and then 
later de-serialisation will 
take too long, and you'd loose the advantages 
of structural sharing along the way. 

**Also,** to serialise/de-serialise sometimes requires special extensions to be 
present for the reader literals, etc.  All too complicated. Best avoided.  

 