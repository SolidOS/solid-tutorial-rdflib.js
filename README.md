# Tutorial for using rdflib.js

This tutorial will walk you through what [rdflib.js](https://github.com/linkeddata/rdflib.js) can do.

## Introduction

`rdflib.js` is a library for fetching, parsing and serializing RDF data
in various formats/serializations (Turtle, RDF/XML, RDFa, etc).

It helps to be familiar with general RDF terminology before working with this
library:

**[RDF Primer](https://www.w3.org/TR/rdf-primer/)**

## Namespace shortcuts

The system uses javascript objects for RDF symbols. Generating them for
predicates is simplified by using a namespace function, turning them into
prefixes:

```javascript
var RDF = Namespace("http://www.w3.org/1999/02/22-rdf-syntax-ns#")
var RDFS = Namespace("http://www.w3.org/2000/01/rdf-schema#")
var FOAF = Namespace("http://xmlns.com/foaf/0.1/")
var XSD = Namespace("http://www.w3.org/2001/XMLSchema#")
```

So now, instead of typing the whole URI for your predicate, i.e.:

```
$rdf.sym('http://xmlns.com/foaf/0.1/knows')
```

You can type:

```
var foafKnows = FOAF('knows');
```

### RDF term types

These are types of nodes in the RDF graph. We call them terms because they are
like terms in a language when you think of RDF as a real language.

| What | How | Term type |
|--------|------|--------|
| Node identified by a URI | `x = $rdf.sym(uri)` | symbol	|
| Blank node | `x = $rdf.bnode()` | 'bnode' |
| Untyped Literal | `x = $rdf.literal('abc')` | literal |
| Typed Literal | `x = $rdf.literal('8080', undefined, XSD('int'))` | literal |
| Literal with language | `x = $rdf.literal('car', 'en')` | literal |
| Ordered list | `x = $rdf.list([node1, node2])` | collection	|

## Creating a data store (graph)

```javascript
var store = $rdf.graph()
```

### Fetch data into the store

You can also load data from an existing URL into the store you just created.

```javascript
var store = $rdf.graph()
var timeout = 5000 // 5000 ms timeout
var fetcher = new $rdf.Fetcher(store, timeout)

fetcher.nowOrWhenFetched(url, function(ok, body, xhr) {
    if (!ok) {
        console.log("Oops, something happened and couldn't fetch data");
    } else {
        // do something with the data in the store (see below)
    }
})
```

## Using data in the store

There are two ways to look at RDF data in a store. You can synchronously use the
`each()` and `any()` methods and `statementsMatching()`, or you can do a query
which returns results asynchronously.

The `each()`, `any()` and `statementsMatching()` take a pattern of subject,
predate, object and source, where for `each()` and `any()` one of s p and o are
undefined and source may be undefined or not. For example, using `$rdf.sym()` to
make an object for an RDF node (symbol),

```javascript
var me = $rdf.sym('https://www.w3.org/People/Berners-Lee/card#i');
var knows = FOAF('knows')
var friend = store.any(me, knows)  // Any one person
```

or using the vocabulary namespace straight away

```javascript
var me = $rdf.sym('http://www.w3.org/People/Berners-Lee/card#i');
var friend = store.any(me, FOAF('friend'))
```

## Wildcards

When one of the terms of the triple is set as *undefined*, it then serves as a
wildcard. In this case, the formula returns the object of a matching triple.

```javascript
var friend = store.any(me, FOAF('age'), undefined)  // Any one age value
```

Alternatively, you get a javascript array of objects using `each()`.

```javascript
var friends = store.each(me, FOAF('knows'), undefined)
for (var i=0; i<friends.length;i++) {
    friend = friends[i]
    console.log(friend.uri) // the WebID of a friend
    ...
}
```

### Wildcards in statements

An alternative to using `each()` comes in the form of `statementsMatching()`,
which returns an array of statements that match a specific triple pattern.

For example, we can obtain all the statements having `foaf:knows` as a
predicate. This way we obtain an array of all the people and all the friends
they have. Then it's up to us to decide what information we want to use, may
that be the subjects or the objects of those statements.

```javascript
var friends = store.statementsMatching(undefined, FOAF('knows'), undefined)
for (var i=0; i<friends.length;i++) {
    friend = friends[i]
    console.log(friend.subject.uri) // a person having friends
    console.log(friend.object.uri) // a friend of a person
    ...
}
```

## Adding data

The `add(s, p, o, w)` method allows a statement to be added to a formula. The
optional `w` argument can be used to keep track of which resource was the source
(URI) for each triple.

```javascript
store.add(me, FOAF('knows'), $rdf.sym('https://fred.me/profile#me')
store.add(me, FOAF('name'), "Albert Bloggs")
```

Note above where the RDF thing in question is a literal, you can just pass a
javascript literal and a string, boolean, or numeric literal of the appropriate
datatype will be used.

The `each()`, `any()` and `statementsMatching()` functions in fact can take a
fourth argument to select data from a particular source. For example, I want to
find **only** the friends in my foaf file, so I force use of my foaf file as the
`w` argument.

```javascript
var myFoafFile = $rdf.sym(uri)
var friendsInMyFOAF = store.each(me, FOAF('knows'), undefined, myFoafFile)
```

## Parsing RDF data

You can use `rdflib.js` to parse arbitrary RDF data.

```javascript
var data
// Fetch data via a regular AJAX call, load from a file, or pass in a literal
// string. In this example, it was loaded from 'https://fred.me/profile'
var store = $rdf.graph()  // Init a new empty graph
var contentType = 'text/turtle'
var baseUrl = 'https://fred.me/profile'
var parsedGraph = $rdf.parse(data, store, baseUrl, contentType)
```
