# cloud-itonami-isco-7312

Open Occupation Blueprint for **ISCO-08 7312**: Musical Instrument Makers and Tuners.

This repository designs a forkable OSS business for a musical-instrument-making/tuning workshop scheduling and logistics coordination practice: a workshop scheduling and supply-coordination robot manages crew/task records under a governor-gated actor, so an instrument-making/tuning crew keeps its own operating records instead of renting a closed workforce-management SaaS.

**Maturity: `:implemented`.** `src/instrumentmaker/` implements the
`InstrumentMakerActor` as a `langgraph.graph/state-graph`
(`instrumentmaker.actor`) wired to an `Instrument Maker Advisor`
(`instrumentmaker.advisor`) and an independent
`InstrumentMakerGovernor` (`instrumentmaker.governor`), following the
itonami actor pattern (ADR-2607121000): `:intake -> :advise -> :govern
-> :decide -+-> :commit (:ok?) +-> :request-approval (:escalate?,
human-in-the-loop interrupt) +-> :hold (:hard?)`. 27 tests / 58
assertions green (`kbb -M:test`). HARD invariants (always hold,
never overridable): worker provenance, workshop provenance,
no-actuation (`:effect` must be `:propose`), a closed op-allowlist
(`:log-work-record`, `:schedule-crew-operation`,
`:flag-safety-concern`, `:coordinate-supply-order` — nothing else may
ever be proposed), and a permanent, unconditional block on any
proposal that would directly finalize an
instrument-construction/tuning-execution decision (e.g. deciding to
proceed with the instrument-construction step or the tuning execution
of a specific instrument) or override a workshop safety officer's
judgment. Always-escalate paths (human sign-off regardless of
confidence, mapping this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)):
`:flag-safety-concern` (always) and `:coordinate-supply-order` above
the registered cost threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a workshop scheduling/logistics coordination robot performs crew scheduling, task/materials-usage/progress-record logging and instrument-materials supply-order coordination for a musical-instrument-making/tuning crew, under an actor that proposes actions and an independent **Instrument Maker Governor** that gates them. The governor never
dispatches hardware itself, never builds or tunes an instrument itself (woodworking/metalworking tools, lacquers, adhesives) on the shop floor, and never finalizes an instrument-construction/tuning-execution decision or overrides a workshop safety officer's judgment; `:high`/`:safety-critical` actions (such as a flagged tool-hazard/fume-exposure concern, or an above-threshold supply order) require human sign-off. **This actor coordinates workshop scheduling/logistics only — it never performs instrument-making/tuning work itself.**

## Core Contract

```text
crew roster + workshop registration + safety-reporting policy
        |
        v
Instrument Maker Advisor -> Instrument Maker Governor -> log/schedule/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, finalize
an instrument-construction/tuning-execution decision, override a workshop
safety officer's judgment, suppress an operating record, or disclose
sensitive data without governor approval and audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `7312`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
