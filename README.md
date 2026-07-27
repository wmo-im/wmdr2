[![Build Status](https://github.com/wmo-im/wmdr2/workflows/validate%20schema%20and%20examples/badge.svg)](https://github.com/wmo-im/wmdr2/actions/workflows/validation.yml)

# wmdr2

WIGOS Metadata Representation 2

* [Terms of Reference](https://github.com/wmo-im/sc-imt/blob/main/et-tt/ad-hoc-tt-wmdr2.adoc)
* Follow work-in-progress: https://github.com/wmo-im/wmdr2-devt

## Schema alignment proposal for observation-series metadata

This branch keeps the modular schema style used in this repository.  The record
schema remains centred on `schemas/wmdrRecordGeoJSON.yaml`, with supporting
schema modules under `schemas/` and external references to OGC API Records,
OGC API Features, and WCMP where appropriate.

The main modelling change is to treat `observingCapabilities` as the
observation-series level of the record.  This keeps the public name proposed in
PR #22, while aligning the contents with the observation-series model being
explored in `wmdr2-devt`:

```text
Facility record
  properties.observingCapabilities[]
    observedProperty / observedFeature / observedGeometry
    observingConfigurations[]
    observingProcedures[]
    reportingProcedures[]

  properties.instruments[]  # reusable instrument catalogue entries
  properties.schedules[]    # reusable observing/reporting schedules
```

The important structural change is that the former `deployments[0..1]` child is
not used by `observingCapability.yaml`.  Its useful content is represented by
`observingConfigurations[]`, because a single observation series may have a
history of methods, sources of observation, instruments, instrument positions,
reference surfaces, vertical distances, or operating status values.

## Facility record

A WMDR2 record is a GeoJSON Feature.  The root `id` is the primary WIGOS Station
Identifier.  Additional WIGOS Station Identifiers are recorded under
`properties.additionalIds`.  The primary facility name is recorded as
`properties.title`; additional facility names or aliases are recorded under
`properties.additionalTitles`.

The facility-level `properties` object may contain:

* `title` and `additionalTitles`
* `additionalIds`
* `description`, `facilityType`, and `wmoRegion`
* reusable OGC `contacts` and contextual `contactAssignments`
* `territory` / `territories` as temporal territory history
* `programAffiliations` as temporal programme affiliation history
* `observingCapabilities` as the observation-series level
* reusable `instruments`
* reusable `schedules`

Only `title` and `observingCapabilities` are required inside the facility
properties object in this proposal.  This avoids making incomplete legacy source
records invalid merely because optional contact, creation-date, instrument, or
link metadata are absent.

## Observing capabilities as observation series

`schemas/observingCapability.yaml` represents a series of observations of a
given property, feature, and observed geometry.  It may include observation-level
contacts, programmes, links, and keywords.

`observingConfigurations[]` documents how observations were made during a
specific configuration period.  Configuration-level fields include:

* `time`
* `observingMethod`
* `operatingStatus` (0..1)
* `sourceOfObservation`
* `instrument` reference
* `serialNumber` (0..1)
* `geometry`
* `referenceSurface`
* `verticalDistanceFromReferenceSurface`

`serialNumber` is therefore instance/configuration metadata, not instrument
catalogue metadata.  `schemas/instrument.yaml` describes reusable instrument
catalogue entries such as manufacturer, model, and observing methods available
from the instrument. This may support an application offering a choice of
instrument by method. The instrument catalogue avoids the need to create a new
instrument catalogue entry for each individual instrument instance.

## Temporal support

`schemas/temporal.yaml` is retained and extended.  It still provides the
`dates` array used by earlier draft structures, and now also provides `time` and
`timePeriod` definitions for the current interval-based model.

Open-ended intervals use `..` as the open bound, for example:

```yaml
time:
  interval:
    - '2020-01-01'
    - '..'
```

## Compatibility notes

This branch avoids a wholesale replacement of the official schema with the
broader WMDR2 development schema.  The intent is a small, modular update that
brings PR #22 closer to the richer observation documentation supported by the
current development examples.
