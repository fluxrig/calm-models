# fluxrig CALM models

CALM (Common Architecture Language Model) documents for the fluxrig payment switch.
The model is a standard document, not a picture: it validates, renders, and stores
in any CALM toolchain.

FINOS hosts the [CALM specification](https://calm.finos.org/) and the
[architecture-as-code](https://github.com/finos/architecture-as-code) repository.

## Contents

```
architectures/
  payment-switch.architecture.json   high-level view: two regions, Racks, scheme uplinks
  rack-east.architecture.json        rack east detail: gears, wires, endpoints
  rack-west.architecture.json        rack west detail: gears, wires, endpoints
flows/
  authorization.flow.json            end-to-end authorization path
  failover.flow.json                 cross-region failover path
```

The overview drills into each Rack through `details.detailed-architecture`,
resolved as a file path in the same `architectures/` directory.

## Validate

Install the CLI:

```shell
npm install -g @finos/calm-cli
```

Validate the architectures from inside `architectures/`:

```shell
cd architectures
calm validate -a payment-switch.architecture.json
calm validate -a rack-east.architecture.json
calm validate -a rack-west.architecture.json
```

All three report 0 errors.

Run the command from inside `architectures/`. The CLI resolves
`details.detailed-architecture` relative to the current working directory, not to
the referencing document. From the repository root the drill targets are reported
as missing.

## Flows

The `calm` CLI does not validate flow documents. `calm validate -a` applies the
architecture rules and reports two errors that do not apply to a flow. The same
two errors appear on the specification's own example flow,
`examples/fluxnova/payment-processing.flow.json`, in the
[finos/architecture-as-code](https://github.com/finos/architecture-as-code)
repository.

Validate a flow against its meta-schema directly:

```shell
calm validate -a flows/authorization.flow.json   # not a flow check
```

Use a JSON Schema validator against `https://calm.finos.org/release/1.2/meta/flow.json`.
Both flows in this repository are schema-valid.

## Render

The CALM Studio web component renders these documents to static SVG at build time.
The generated diagrams are published with the
[architecture as code entry](https://antoniuk.org/posts/architecture-as-code-calm-finos/).

## Consume from a CALM Hub

A CALM Hub in `github` storage mode reads this repository directly. Point it at the
namespace:

```properties
calm.database.mode=github
calm.github.namespaces=fluxrig|fluxrig/calm-models|main
```

The Hub clones the repository and serves every `architectures/` and `flows/` file as
a versioned resource. A version is the commit SHA that last changed the file.

The public Hub at [hub.calm.finos.org](https://hub.calm.finos.org/) is read-only and
accepts no external namespaces.

## Source

Generated from the fluxrig scenario file `payment_switch_two_regions.yaml`. Wiring
the generator into the `fluxrig` build is planned.

## License

Apache-2.0.
