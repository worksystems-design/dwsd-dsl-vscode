# Flight Route DSL — Syntax Reference

Every flight-route construct, with a minimal example. Each example is a complete, parse-clean `*.dwsd-flightroute` file — the same bytes shipped under `examples/route/`.

## edge vocabulary (generate/copy/feedback)

The three canonical edges in one `path:` — `generate` (`->`, solid, same band), `copy` (`-->`, dashed, cross-band), and `feedback` (`..>`, dotted).

```dwsd.flightroute
# edges — all three canonical edges in one route: generate (solid), copy (dashed), feedback (dotted).
title: Edge Vocabulary
for: feature
bands:
  - name: Strategy
    fl: 3
  - name: Delivery
    fl: 1
path:
  - epic @ Strategy -> story @ Strategy
  - story @ Strategy --> task @ Delivery
  - task @ Delivery ..> epic @ Strategy
```

## multiple bands (one per flight level)

Multiple `bands:` records, one named band per flight level (3/2/1); stages live across the named bands and hop between them along the `path:`. A single band may also span several levels via `fl: [1, 2]` (see the reference).

```dwsd.flightroute
# bands — one named band per flight level; stages live across named bands.
title: Multi-Band Route
for: feature
bands:
  - name: Strategy
    fl: 3
  - name: Coordination
    fl: 2
  - name: Delivery
    fl: 1
path:
  - vision @ Strategy --> epic @ Coordination
  - epic @ Coordination --> story @ Delivery
```

## outside-band delivery (() / ⊙)

A `copy` into the ship band, then a dotted `..> ()` terminating at the reserved outside-band `()` / ⊙ sink.

```dwsd.flightroute
# delivery — a copy into the delivery band, then a dotted `..> ()` to the reserved outside-band sink.
title: Delivery Route
for: feature
bands:
  - name: Build
    fl: 2
  - name: Ship
    fl: 1
path:
  - account @ Build --> account @ Ship
  - account @ Ship ..> ()
```

## path & bands

The minimal complete route: a `bands:` declaration plus a `path:` of one `generate` edge (`->`) between two stages in the `itemType @ Band` form.

```dwsd.flightroute
# minimal — the smallest complete route: one band, one generate edge between two stages.
title: Minimal Route
for: feature
bands:
  - name: Delivery
    fl: 1
path:
  - epic @ Delivery -> story @ Delivery
```

## route → board binding (bound-to:)

A `bound-to:` layer maps a route band onto a concrete board and its stages via the shared `[[board]]` locator, keeping the `path:` itself unbound.

```dwsd.flightroute
# binding — a `bound-to:` layer maps a route band to a concrete board via the shared locator.
title: Bound Route
for: feature
bands:
  - name: Coordination
    fl: 2
  - name: Delivery
    fl: 1
path:
  - epic @ Coordination --> story @ Delivery
bound-to:
  - band: Delivery
    boards:
      - board: "[[team-alpha]]"
        stages:
          - story: "lane:Frontend/col:Doing"
```

## route triggers

A plain valid route with external `triggers:` that generate stages and a `path:`; a companion entity can anchor to one of its stages host-side.

```dwsd.flightroute
# refs — a plain valid route; a companion entity anchors to one of its stages host-side (see anchors.md).
title: Referenced Route
for: feature
bands:
  - name: Coordination
    fl: 2
  - name: Delivery
    fl: 1
triggers:
  - name: new feature idea
    generates:
      - epic @ Coordination
path:
  - epic @ Coordination --> story @ Delivery
```
