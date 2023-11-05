## Relational Model Versus Document Model
### The Birth of NoSQL
In the 2010s, NoSQL is the latest attempt to overthrow the relational model’s dominance.
There are several driving forces behind the adoption of NoSQL databases, including:
- A need for greater scalability than relational databases can easily achieve, including very large datasets or very high write throughput

### The Object-Relational Mismatch
If data is stored in relational tables, an awkward translation layer is required between the objects in the application code and the database model of tables, rows, and columns.

Object-relational mapping (ORM) frameworks like ActiveRecord and Hibernate reduce the amount of boilerplate code required for this translation layer, but they can’t completely hide the differences between the two models.
Some developers feel that the JSON model reduces the impedance mismatch between the application code and the storage layer.
The JSON representation has better locality than the multi-table schema in Figure 2-1. If you want to fetch a profile in the relational example, you need to either perform multiple queries (query each table by user_id) or perform a messy multi-way join between the users table and its subordinate tables. In the JSON representation, all the relevant information is in one place, and one query is sufficient.

### Many-to-One and Many-to-Many Relationships
Use ID, not name attribute to store data.
The advantage of using an ID is that because it has no meaning to humans, it never needs to change: the ID can remain the same, even if the information it identifies changes.
In relational databases, it’s normal to refer to rows in other tables by ID, because joins are easy. In document databases, joins are not needed for one-to-many tree structures, and support for joins is often weak.

### Are Document Databases Repeating History?
Network Model(CODASYL): Basically a huge graph with nodes and edges, and if were to query, had to follow and `access path`.

Relational Model: Layout all the data in the open (table, rows), and let the query optimizer do the search.

### Relational Versus Document Database Today
In favor of the document models are
- schema flexibility
- better performance due to locality
- If my application has a document structure(i.e., a tree of one-to-many relationships, where typically the entire tree is loaded at once),

Limitations of document models is that you cannot refer directly to a nested item within a document, but instead you need to say something like “the second item in the list of positions for user 251”.
If your application does use many-to-many relationships, the document model becomes less appealing. It’s possible to reduce the need for joins by denormalizing, but then the application code needs to do additional work to keep the denormalized data consistent.

In favor of relational modesl are
- better support for joins, many-to-one, many-to-many relationships

## Query Language for Database
Declarative vs Imperative

Declarative: Just specify what the results must meet, and how you want the data to be transformed, **but not how to achieve the goal**
```sql
-- sharks  =  σ_family = “Sharks” (animals), 
SELECT * FROM animals WHERE family = 'Sharks';
```

Imperative: Specify how to achieve the gaol

```javascript
function getSharks() {
    var sharks = [];
    for (var i = 0; i < animals.length; i++) {
        if (animals[i].family === "Sharks") {
            sharks.push(animals[i]);
        }
    }
    return sharks;
}
```

Declarative Query Language is attractive because
- more concise and easier to work with than an imperative API.
- hides implementation of the database engine, which makes it possible for the database system to introduce performance improvements without requiring any changes to queries
- lend themselves to parallel execution. Imperative code is very hard to parallelize across multiple cores and multiple machines, because it specifies instructions that must be performed in a particular order.

### Declarative Queries on the Web
Declarative:
```css
li.selected > p {
    background-color: blue;
}
```
Imperative:

```javascript
var liElements = document.getElementsByTagName("li");
for (var i = 0; i < liElements.length; i++) {
    if (liElements[i].className === "selected") {
        var children = liElements[i].childNodes;
        for (var j = 0; j < children.length; j++) {
            var child = children[j];
            if (child.nodeType === Node.ELEMENT_NODE && child.tagName === "P") {
                child.setAttribute("style", "background-color: blue");
            }
        }
    }
}
```
### MapReduce Querying
MapReduce is a programming model for processing large amounts of data in bulk across many machines, popularized by Google.
A limited form of MapReduce is supported by some NoSQL datastores, including MongoDB and CouchDB, as a mechanism for performing read-only queries across many documents.

MapReduce is neither a declarative query language nor a fully imperative query API, but somewhere in between: the logic of the query is expressed with snippets of code, which are called repeatedly by the processing framework. It is based on the map (also known as collect) and reduce (also known as fold or inject) functions that exist in many functional programming languages.

#### Map Reduce for Shark Report
Imagine you are a marine biologist, and you add an observation record to your database every time you see animals in the ocean. Now you want to generate a report saying how many sharks you have sighted per month.

In `PostgreSQL`, you might express the query like this
```sql
SELECT date_trunc('month', observation_timestamp) AS observation_month, 
       sum(num_animals) AS total_animals
FROM observations
WHERE family = 'Sharks'
GROUP BY observation_month;
```
The `date_trunc('month', timestamp)` function determines the calendar month containing timestamp, and returns another timestamp representing the beginning of that month.

This query first filters the observations to only show species in the Sharks family, then groups the observations by the calendar month in which they occurred, and finally adds up the number of animals seen in all observations in that month.

In `MongoDB`, it can be represented as
```javascript
db.observations.mapReduce(
    function map() { // 2. called once for every document that matches query
        var year  = this.observationTimestamp.getFullYear();
        var month = this.observationTimestamp.getMonth() + 1;
        emit(year + "-" + month, this.numAnimals); // 3. emit a key, such as '2013-12' and a value (# of animals)
    },
    function reduce(key, values) { // 4. key-value pairs emitted by map are grouped by key
        return Array.sum(values); // 5. aggregation
    },
    {
        query: { family: "Sharks" }, // 1. filter to consider only shark species can be specified declaratively
        out: "monthlySharkReport" // 6. final output
    }
);
```

In later features of `MongoDB` (v2.2), they added support for a declarative language called `aggregation pipeline`.
```javascript
db.observations.aggregate([
    { $match: { family: "Sharks" } },
    { $group: {
        _id: {
            year:  { $year:  "$observationTimestamp" },
            month: { $month: "$observationTimestamp" }
        },
        totalAnimals: { $sum: "$numAnimals" }
    } }
]);
```

## Graph-Like Data Models
Earlier we saw that document model is appropriate if your application has mostly one-to-many relationships (tree-structured data) or no relationships between records. \
But what if many-to-many relationships are very common in your data? The relational model can handle simple cases of many-to-many relationships, but as the connections within your data become more complex, it becomes more natural to start modeling your data as a **graph**.

Graphs consist of 
- vertices(nodes)
- edges

These are not necessarily **homogenous**. For example, Facebook maintains a single graph with many different types of vertices and edges: vertices represent people, locations, events, checkins, and comments made by users; edges indicate which people are friends with each other, which checkin happened in which location, who commented on which post, who attended which event, and so on.

In the remaining chapter, concepts will be illustrated by an example in the figure below. It shows two people, Lucy from Idaho and Alain from Beaune, France. They are married and living in London.

![](/assets/techlog/distributed_system/designing_data_intensive_applications/ch02/graph_structured_data_example.png)

### Property Graphs

In poperty graph model (implemented by Neo4j, Titan, and InfiniteGraph),

Each `vertex` consits of
- A unique identifier
- A set of outgoing edges
- A set of incoming edges
- A collection of properties (key-value pairs)

Each `edge` consists of
- A unique identifier
- The tail vertex
- The head vertex
- A label to describe the kind of relationship between the two vertices
- A collection of properties (key-value pairs)

Some important aspects are
- Any vertex can have an edge connecting it with any other vertex.
- Given any vertex, you can efficiently find both its incoming and its outgoing edges, and thus traverse the graph.
- By using different labels for different kinds of relationships, you can store several different kinds of information in a single graph, while still maintaining a clean data model.

Graphs are good for evolvability: as you add features to your application, a graph can easily be extended to accommodate changes in your application’s data structures.

### The Cypher Query Language
`Cypher` is a declarative query language for property graphs, created for the `Neo4j` graph database

Each vertex is given a symbolic name like `USA` or `Idaho`, and other parts of the query can use those names to create edges bewteen the vertices, using an arrow notation.
`(Idaho) -[:WITHIN]-> (USA)` creates an edge labeled `WITHIN`, with Idaho as the tail node and USA as the head node.

Below is the cypher query language to populate the example graph data shown before.
```cypher
CREATE
  (NAmerica:Location {name:'North America', type:'continent'}),
  (USA:Location      {name:'United States', type:'country'  }),
  (Idaho:Location    {name:'Idaho',         type:'state'    }),
  (Lucy:Person       {name:'Lucy' }),
  (Idaho) -[:WITHIN]->  (USA)  -[:WITHIN]-> (NAmerica),
  (Lucy)  -[:BORN_IN]-> (Idaho)
```

Once we have the graph created, we can then start asking interesting questions: *find the names of all the people who emigrated from the United States to Europe*

To be more precise, here we want to find all the vertices that have a `BORN_IN` edge to a location within the `USA`, and also a `LIVING_IN` edge to a location within `Europe`, and return the name property of each of those vertices.

The same arrow notation is used in a `MATCH` clause to find patterns in the graph: `(person) -[:BORN_IN]-> ()` matches any two vertices that are related by an edge labeled `BORN_IN`.

```cypher
MATCH
  (person) -[:BORN_IN]->  () -[:WITHIN*0..]-> (us:Location {name:'United States'}),
  (person) -[:LIVES_IN]-> () -[:WITHIN*0..]-> (eu:Location {name:'Europe'})
RETURN person.name
```
The above query can be read as follows:

- `Person` has an outgoing `BORN_IN` edge to some vertex. From that vertex, you can follow a chain of outgoing `WITHIN` edges until eventually you reach a vertex of type `Location`, whose name property is equal to `"United States"`.
- That same person vertex also has an outgoing `LIVES_IN` edge. Following that edge, and then a chain of outgoing `WITHIN` edges, you eventually reach a vertex of type `Location`, whose name property is equal to `"Europe"`.

### Graph Queries in SQL
If we put graph data in a relational structure, can we also query it using SQL?
The answer is yes, but with some difficulty.

To implement the above query, we can use the recursive common table expressions. However, the syntax is very clumsy in comparison to Cypher.

```sql
WITH RECURSIVE

  -- in_usa is the set of vertex IDs of all locations within the United States
  in_usa(vertex_id) AS (
      SELECT vertex_id FROM vertices WHERE properties->>'name' = 'United States' 
    UNION
      SELECT edges.tail_vertex FROM edges 
        JOIN in_usa ON edges.head_vertex = in_usa.vertex_id
        WHERE edges.label = 'within'
  ),

  -- in_europe is the set of vertex IDs of all locations within Europe
  in_europe(vertex_id) AS (
      SELECT vertex_id FROM vertices WHERE properties->>'name' = 'Europe' 
    UNION
      SELECT edges.tail_vertex FROM edges
        JOIN in_europe ON edges.head_vertex = in_europe.vertex_id
        WHERE edges.label = 'within'
  ),

  -- born_in_usa is the set of vertex IDs of all people born in the US
  born_in_usa(vertex_id) AS ( 
    SELECT edges.tail_vertex FROM edges
      JOIN in_usa ON edges.head_vertex = in_usa.vertex_id
      WHERE edges.label = 'born_in'
  ),

  -- lives_in_europe is the set of vertex IDs of all people living in Europe
  lives_in_europe(vertex_id) AS ( 
    SELECT edges.tail_vertex FROM edges
      JOIN in_europe ON edges.head_vertex = in_europe.vertex_id
      WHERE edges.label = 'lives_in'
  )

SELECT vertices.properties->>'name'
FROM vertices
-- join to find those people who were both born in the US *and* live in Europe
JOIN born_in_usa     ON vertices.vertex_id = born_in_usa.vertex_id 
JOIN lives_in_europe ON vertices.vertex_id = lives_in_europe.vertex_id;
```

### Triple-Stores and SPARQL
The `triple-store model` is mostly equivalent to the property graph model, using different words to describe the same ideas.
In a triple-store, all information is stored in the form of very simple three-part statements: (subject, predicate, object).
```
@prefix : <urn:example:>.
_:lucy     a       :Person.
_:lucy     :name   "Lucy".
_:lucy     :bornIn _:idaho.
_:idaho    a       :Location.
_:idaho    :name   "Idaho".
_:idaho    :type   "state".
_:idaho    :within _:usa.
_:usa      a       :Location.
_:usa      :name   "United States".
_:usa      :type   "country".
_:usa      :within _:namerica.
_:namerica a       :Location.
_:namerica :name   "North America".
_:namerica :type   "continent".
```

`SPARQL` is a query language for triple-stores using the RDF data model. It predates Cypher, and since Cypher’s pattern matching is borrowed from SPARQL, they look quite similar
```sparql
PREFIX : <urn:example:>

SELECT ?personName WHERE {
  ?person :name ?personName.
  ?person :bornIn  / :within* / :name "United States".
  ?person :livesIn / :within* / :name "Europe".
}
```

### The Foundation: Datalog

Datalog is a much older language than `SPARQL` or `Cypher`. In practice, Datalog is used in a few data systems: for example, it is the query language of Datomic [40], and Cascalog [47] is a Datalog implementation for querying large datasets in Hadoop.

Data is defined in such way.
```
name(namerica, 'North America').
type(namerica, continent).

name(usa, 'United States').
type(usa, country).
within(usa, namerica).

name(idaho, 'Idaho').
type(idaho, state).
within(idaho, usa).

name(lucy, 'Lucy').
born_in(lucy, idaho).
```

`Cypher` and `SPARQL` jump in right away with `SELECT`, but Datalog takes a small step at a time. We define rules that tell the database about new predicates: here, we define two new predicates, within_recursive and migrated.
```
within_recursive(Location, Name) :- name(Location, Name).     /* Rule 1 */

within_recursive(Location, Name) :- within(Location, Via),    /* Rule 2 */
                                    within_recursive(Via, Name).

migrated(Name, BornIn, LivingIn) :- name(Person, Name),       /* Rule 3 */
                                    born_in(Person, BornLoc),
                                    within_recursive(BornLoc, BornIn),
                                    lives_in(Person, LivingLoc),
                                    within_recursive(LivingLoc, LivingIn).

?- migrated(Who, 'United States', 'Europe').
```

A rule applies if the system can find a match for all predicates on the righthand side of the `:-` operator. When the rule applies, it’s as though the lefthand side of the `:-` was added to the database (with variables replaced by the values they matched).

One possible way of applying the rules is thus:
1. `name(namerica, 'North America')` exists in the database, so rule 1 applies. It generates `within_recursive(namerica, 'North America')`. 
2. `within(usa, namerica)` exists in the database and the previous step generated `within_recursive(namerica, 'North America')`, so rule 2 applies. It generates `within_recursive(usa, 'North America')`.
3. `within(idaho, usa)` exists in the database and the previous step generated `within_recursive(usa, 'North America')`, so rule 2 applies. It generates `within_recursive(idaho, 'North America')`.

With these rules defined, we can determine that Idaho is in North America.
