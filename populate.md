# Populate-exchange

This [[exchange]] allows you to automatically populate the selection set for a mutation operation, This
selection will be based on prior queries, this way we can efficiently refresh cached data.

## Refactor

This refactor is based on the following [RFC](https://github.com/FormidableLabs/urql/issues/964) and
aims at easing the transition from a [[document cache]] to a [[normalized cache]].

### Goals

- We need to be able to populate [[Field arguments]]
- We need to be able to populate the [[viewer field]] (`mutation addTodo { viewer @populate }`)
- We need to be able to add all queries as a separate selection to refetch queries (`mutation addTodo @populate`)

## Architecture

Initially when a [[Query]] [[operation]] comes in we need to traverse all [[operation definitions]].
These [[operation definitions]] will have [[selections]] containing [[fields]] which in turn can have
more [[selections]]. These fields can potentially contain [[arguments]] which can be contextualized within
the current [[operation]] which will contain [[variables]] for these [[arguments]].

We need to have an accessible and performant structure that allows us to follow a set of types with their optional arguments and variables
to the fields currently used within the application.

When we enter a mutation we now have a few possible cases that we need to  go through:

- **Classic** `mutation { addTodo @populate }`
- **Viewer** `mutation { addTodo { viewer @populate} }`
- **Queries** `mutation @populate { addTodo }`

We need to assess which case we are in and start the correct procedure.

### Classic

We find out what type is used inside of the [[Mutation]] and look in our data-structure what fields we've
used for this type. When we figure this out we can start creating [[Fragments]] to populate the selection of
this [[Mutation]].

An uncertainty that remains here is what happens if one of these fields has an [[argument]] that's used with
two different values? Should we create two distinct aliases or is this actually not possible?

My gut would say that  this is only solvable with the [[Queries]] approach.

### Viewer

Linear to the [[Classic]] approach we'll need to see if our [[Viewer]] field contains a fields
of the intersecting type and populate these with [[Fragments]].

### Queries

This is a bit of a harder scenario when we reach the point of bigger applications, for instance when we would
have a list of items and we've entered the details screen for multiple ones then we'll have a huge list of queries
to refetch.

This means that we'll probably have to find a way to identify whether or not this [[Mutation]] is targetted as a create/delete/update,
with an update we can run the details query with the correct id. For delete/create we can be sure that we need to focus on the active list Query.
