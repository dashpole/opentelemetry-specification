# Telemetry Stability

**Status**: [Development](document-status.md)

<details>
<summary>Table of Contents</summary>

<!-- START doctoc -->

- [Unstable Instrumentations](#unstable-instrumentations)
- [Stable Instrumentations](#stable-instrumentations)
  * [Development Telemetry in Stable Instrumentations](#development-telemetry-in-stable-instrumentations)
  * [Fixed Schema Telemetry Producers](#fixed-schema-telemetry-producers)
  * [Schema-File Driven Telemetry Producers](#schema-file-driven-telemetry-producers)

<!-- END doctoc -->

</details>

This section defines stability requirements for telemetry produced by
OpenTelemetry instrumentations.

All OpenTelemetry-authored instrumentations are labeled to be either `Unstable` or `Stable`
from the perspective of the telemetry they produce.

Adding of new metrics, spans, span events or log records and adding of
new attributes to spans, span events, log records or resources are considered
additive, non-breaking changes and are always allowed for `Unstable` and `Stable`
instrumentations.

Other changes in the produced telemetry are regulated by the following rules.

Changes in emitted telemetry that satisfy the
[security-related changes exception](versioning-and-stability.md#security-related-changes)
MAY be made without incrementing the major version, even when the stability rules
below would otherwise prohibit them. This exception applies to both fixed-schema
and schema-file driven producers, including during the moratorium on relying on
schema transformations. All conditions of the exception apply, including its
scope, compatibility guarantees, and release note requirements.

## Unstable Instrumentations

Unstable telemetry-producing instrumentations (unstable instrumentations for short) SHOULD
be clearly labeled so by any means the instrumentations authors consider idiomatic for
their language, e.g. via version numbers, artifact names, documentation, etc.

Unstable instrumentations provide no guarantees about the shape of
the telemetry they produce and how that shape changes over time, from version to version.
Span or metric names, attributes of any telemetry items may change without any
restrictions. The produced telemetry MAY specify a Schema URL if the telemetry data
conforms to a particular Schema.

Unstable instrumentations authored by OpenTelemetry MAY produce additional telemetry that
is not described by OpenTelemetry semantic conventions.

TODO: decide if it is necessary to indicate on the wire if the produced telemetry is
coming from an unstable instrumentation.

## Stable Instrumentations

> [!WARNING]
> There is a moratorium on relying on schema transformations for telemetry stability.

Stable telemetry-producing instrumentations (stable instrumentations for short) SHOULD
be clearly labeled so by any means the instrumentations authors consider idiomatic for
their language, e.g. via version numbers, artifact names, documentation, etc.

Stable instrumentations fall into 2 categories: fixed-schema producers and schema-file
driven producers.

Stable instrumentations authored by OpenTelemetry SHOULD NOT produce telemetry with
`Stability = Stable` (the default when unspecified) that is not described by stable
OpenTelemetry semantic conventions. If, however, this rule is broken the
instrumentations MUST NOT change such telemetry, regardless of whether they
are fixed-schema producers or schema-file driven producers. Once the produced telemetry
is added to the semantic conventions, changes will be allowed as described below.

### Development Telemetry in Stable Instrumentations

A `Stable` instrumentation MAY include telemetry marked with `Stability = Development`
via the [Tracing API](trace/api.md#span-creation),
[Metrics API](metrics/api.md#instrument-advisory-parameter-stability), or
[Logs API](logs/api.md#emit-a-logrecord) (for example, telemetry conforming to
`development` OpenTelemetry semantic conventions).

- Telemetry emitted with `Stability = Stable` (the default when unspecified) by a
  `Stable` instrumentation remains bound by the stability requirements for
  [Fixed Schema Telemetry Producers](#fixed-schema-telemetry-producers) and
  [Schema-File Driven Telemetry Producers](#schema-file-driven-telemetry-producers)
  below.
- A `Stable` instrumentation MAY modify, rename, or remove telemetry marked with
  `Stability = Development` without incrementing the major version.
- Graduating a piece of telemetry from `Development` to `Stable` (so that it becomes
  emitted by default when the SDK provider's `minimum_stability` is `Stable`) is a
  non-breaking additive change permitted in a minor version release of a `Stable`
  instrumentation. Conversely, lowering the stability level of existing `Stable`
  telemetry to `Development` (which suppresses it by default) is a breaking change
  that requires a major version bump.

### Fixed Schema Telemetry Producers

Instrumentations that are labeled `Stable` and do not include the Schema URL in the
produced telemetry are called Fixed Schema Telemetry Producers.

Such instrumentations are prohibited from changing any produced telemetry. If the
specification changes over time and the semantic conventions are updated, the
instrumentation is still prohibited from adopting the changes. If the instrumentation
wishes to adopt the semantic convention changes it must first become a
[Schema-File Driven Telemetry Producer](#schema-file-driven-telemetry-producers) by
adding an appropriate Schema URL in the produced telemetry.

### Schema-File Driven Telemetry Producers

Stable instrumentations that include the Schema URL in the produced telemetry are
called Schema-File Driven Telemetry Producers.

Such instrumentations are prohibited from changing the produced telemetry until
the moratorium on relying on schema transformations for telemetry stability is lifted
and until that date are subject to exactly the same restrictions as
[Fixed Schema Telemetry Producers](#fixed-schema-telemetry-producers).

After the moratorium is lifted, stable instrumentations are allowed to change the produced telemetry
if all the following conditions are fulfilled:

- The change is part of OpenTelemetry semantic conventions and is in a released
  version of the specification.
- The change has a corresponding [published](schemas/README.md#opentelemetry-schema)
  OpenTelemetry Schema File that describes the change.
- The produced telemetry correctly specifies the respective Schema URL.
