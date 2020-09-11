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
