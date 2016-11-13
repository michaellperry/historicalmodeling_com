---
layout: page
title: The Factual Modeling Language
permalink: /Factual/
---

The Factual modeling language describes a historical model as a collection if interrelated facts. Factual compilers produce code for libraries like UpdateControls.Correspondence. The rules of Factual are:

- A fact has key fields, which are immutable.
- The identity of the fact is defined by the values of the key fields. If all key fields are equal, the facts are the same.
- If a key field is a reference to another fact, that fact is called a predecessor. The fact containing the reference is called a successor.
- A fact has mutable fields, which are not part of the key and do not define predecessors.
- A fact has queries.
- Queries find facts by their predecessor/successor relationships.

A Factual file represents a namespace -- a self-contained collection of interrelated facts.

```
factual_file -> "namespace" dotted_identifier ";" import* fact*
```

A Factual file may import other namespaces. When doing so, it explicitly aliases the facts that it is importing. If one identifier is given, it is both the local and true name of the fact. If two are given, the identifier on the left is the local name, and the identifier on the right is the true name.

```
import -> "import" dotted_identifier  "{" alias* "}
alias -> identifier ("=" identifier)? ";"
```

The remainder of a Factual file consists of facts. A fact contains three sections:

- `key` - Required. Contains all key fields.
- `mutable` - Optional. Contains all mutable fields.
- `query` - Optional. Contains all queries and predicates.

The key section may contain the `unique` modifier. If it does, then each instance of this fact is unique, even if all other key fields are equal. The mutable section may contain the `delete` and `undelete` modifiers. These allow the fact to be deleted and undeleted.

```
fact -> "fact" identifier "{" key_section mutable_section? query_section? "}"
key_section -> "key" ":" ("unique" ";")? field*
mutable_section -> "mutable" ":" ("delete" ";" ("undelete" ";")?)? field*
query_section -> "query" ":" (query | predicate)*
```

A field is a named value or fact reference. Key fields *cannot* change. Mutable fields *can* change.

```
field -> simple_field | publish_field
simple_field -> type identifier ";"
publish_field -> "publish" type identifier publish_condition? ";"
publish_clause -> "not"? "this" "." identifier
publish_condition -> "where" clause ("and" clause)*
```

The type of a field can be a native data type, a reference to another fact, or a structure. Structures are declared in place and are not named. A type also expresses cardinality:

- `?` - Zero or one
- `*` - Zero to many
- *neither* - One

```
type -> (native_type | identifier | structure) ("?" | "*")?
native_type -> "int" | "float" | "char" | "string" | "date" | "time"
structure -> "(" type identifier ("," type identifier)* ")"
```

The `publish` keyword is allowed on key fields that are references to other facts, and any mutable field. For a key field, it indicates that the fact being defined (the successor) is published to the fact being referenced (the predecessor). Any client in the community that subscribes to the predecessor will receive the successor. They will also receive all predecessors, direct and indirect, of the successor. They will receive all successors, direct or indirect, of the successor that are not themselves published.

For a mutable field, it indicates that the value of the mutable field is published to the fact being defined. Subscribers to the fact will receive the value. If the field is a reference to another fact, then subscribers will receive that fact and all direct and indirect predecessors.

A publish clause may have a condition. If so, subscribers will receive that successor only until a fact is added that causes that condition to become false. Thereafter, they will not receive that successor for that publication, even if the condition again becomes true.

A query is a list of sets that describe related facts. A set is of the form: "All *facts* of type *type* such that *fact*.*field* = *other fact*". The first set in the list relates facts to `this` -- the scope of the query. Each subsequent set relates facts to the prior set. The final set determines the result of the query.

```
query -> identifier "*" identifier "{" set+ "}"
set -> identifier identifier ":" path "=" path condition?
path -> dotted_identifier | "this"
clause -> "not"? identifier "." identifier
condition -> "where" clause ("and" clause)*
```

A predicate determines whether a set is empty or not. Predicates can be used in queries.

```
predicate -> "bool" identifier "{" "not"? "exists" set + "}"
```