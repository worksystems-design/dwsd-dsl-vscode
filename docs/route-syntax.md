# Flight Route DSL — Syntax Reference

Every flight-route construct, with a minimal example. Each example is a complete, parse-clean `*.dwsd-flightroute` file — the same bytes shipped under `examples/route/`.

## activity arrow

An annotated edge between flow nodes: `-[generate]->`, `-[copy]->`, `-[feedback]->`, optionally typed `-[generate: @item]->`.

```dwsd.flightroute
route "Activity Arrow"
flow:
    @discovery
      -[generate]-> @delivery
```

## bands:

Custom flight-level bands declaration with id, label, and levels: `bands:` then `- id: … label: … levels: […]`.

```dwsd.flightroute
route "Custom Bands"
bands:
  - id: strategy label: "Strategy" levels: [3]
  - id: delivery label: "Delivery" levels: [1] default
```

## flow:

The flow block: indented flow nodes (4 spaces) describing how items move between systems.

```dwsd.flightroute
route "Simple Flow"
flow:
    @backlog
    @discovery
```

## for: item type

Declares the primary item type the route describes: `for: @feature`.

```dwsd.flightroute
route "Idea Pipeline"
for: @feature
```

## parallel branch

A parallel (6-space-indented) activity-arrow branch off a flow node, including the `-[deliver]` terminator.

```dwsd.flightroute
route "Parallel Branch"
flow:
    @discovery
      -[generate]-> @build
      -[deliver]
```

## path:

The flight-level sequence the work travels through: `path: FL3: Strategy -> FL2: Coordination -> FL1: Delivery`.

```dwsd.flightroute
route "Flight Levels"
path: FL3: Strategy -> FL2: Coordination -> FL1: Delivery
```

## route name

The optional quoted name of a flight route: `route "Feature Delivery"` (or bare `route`).

```dwsd.flightroute
route "Feature Delivery"
```

## triggers:

The comma-separated list of events that kick the route off: `triggers: new idea, customer request`.

```dwsd.flightroute
route "Intake"
triggers: new idea, customer request
```

## typed-path flow ref

A flow node referencing a board location, either `@board#column` or a typed path `@board#lane:Name/col:Name`.

```dwsd.flightroute
route "Typed Path Reference"
flow:
    @team-board#lane:Team Alpha/col:Doing
```
