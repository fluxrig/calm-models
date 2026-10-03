# fluxrig CALM models

CALM (Common Architecture Language Model) documents for the fluxrig payment switch.
The model is a standard document, not a picture: it validates, renders, and stores
in any CALM toolchain.

These documents describe [fluxrig](https://github.com/jaab-tech/fluxrig) — an open-source
ISO 8583 payment switch made of two binaries: the **Rack**, an edge node that runs
gears and processes the traffic, and the **Mixer**, the central control plane. The
model here covers a two-region deployment: one Rack per region, the scheme uplinks
each Rack reaches, and the failover path between them.

- Source code: [jaab-tech/fluxrig](https://github.com/jaab-tech/fluxrig)
- Worked example: [Building the payment-switch Conductor](https://fluxrig.org/docs/tutorials/payment-switch-conductor)
- Documentation: [fluxrig.org/docs](https://fluxrig.org/docs/)
- Article: [Architecture as Code Needs a Standard, Not Only a Good Tool](https://antoniuk.org/posts/architecture-as-code-calm-finos/)

CALM itself is hosted by FINOS: the [specification](https://calm.finos.org/), the
[reference](https://calm.finos.org/reference/), and the
[architecture-as-code](https://github.com/finos/architecture-as-code) repository.

## Why this repository exists

The write-up [Architecture as Code Needs a Standard, Not Only a Good Tool](https://antoniuk.org/posts/architecture-as-code-calm-finos/)
explains what was done here and why: how the fluxrig payment switch was modelled in
CALM, how the documents validate and render, how the decorators removed the node
duplication, and what CALM still leaves to the tool. Read it first for the full
context behind these files.

## Contents

```
architectures/
  payment-switch.architecture.json   high-level view: two regions, Racks, scheme uplinks
  rack-east.architecture.json        rack east detail: gears, wires, endpoints
  rack-west.architecture.json        rack west detail: gears, wires, endpoints
decorators/
  fluxrig-deployment.decorator.json  region, Rack, and endpoint facts
  fluxrig-gear-placement.decorator.json   gear host, type, and wire facts
flows/
  authorization.flow.json            end-to-end authorization path
  failover.flow.json                 cross-region failover path
```

The overview drills into each Rack through `details.detailed-architecture`,
resolved as a file path in the same `architectures/` directory.

## The model in fluxrig terms

| CALM element | fluxrig meaning |
|---|---|
| `fluxrig:region` | A deployment region. Hosts one Rack, and the Mixer in the east. |
| `fluxrig:rack` | A Rack: the edge node that runs gears and processes traffic. |
| `fluxrig:mixer` | The Mixer: the central control plane, hosted in region east. |
| `fluxrig:conductor` | The gear that correlates a request with its response and routes it to a scheme. |
| `fluxrig:codec-iso8583` | A gear that encodes and decodes ISO 8583 messages. |
| `fluxrig:io-iso8583` | A gear that reads and writes ISO 8583 over a socket. |
| `fluxrig:bento` | A gear that transforms messages with Bento. |
| `actor` | POS terminals, the external systems that submit authorizations. |
| `system` | Scheme endpoints, the external counterparties. |

The `fluxrig:` node types are an extension the CALM schema permits. The Conductor,
its correlation key, its routes, and its ticket TTL are described in the
[payment-switch Conductor tutorial](https://fluxrig.org/docs/tutorials/payment-switch-conductor),
and the gear reference is at [fluxrig.org/docs/reference/gears](https://fluxrig.org/docs/reference/gears/overview).

## Decorators

A node that appears in more than one document used to carry its deployment facts
in every copy. The decorators hold those facts once and apply them by `unique-id`.

- `fluxrig-deployment.decorator.json` owns the region, Rack, Mixer, and external
  endpoint facts: the socket of each scheme and terminal, the Rack region, and the
  transport between Racks.
- `fluxrig-gear-placement.decorator.json` owns the placement and wire facts of every
  gear: the Rack that hosts it, its gear type, socket, mode, encoding, TLS mode, and
  the Conductor timers.

Nine nodes had their facts copied across documents. The decorators removed that
duplication: 94 metadata keys now live once. A socket or gear change is one edit in
one file, and every architecture document that names the node sees it.

The node keeps its identity, name, description, type, and presentation hint. The
decorator is the single source for the facts shared across documents. Metadata that
remains in a node is a presentation hint (`icon`, `labels`), not a deployment fact.

Validate a decorator against its schema with a JSON Schema validator, since the
`calm` CLI does not have a decorator check:

```shell
# schema: https://calm.finos.org/release/1.2/meta/decorators.json#/defs/decorator
```

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

The Hub clones the repository and serves every `architectures/`, `decorators/`, and
`flows/` file as a versioned resource. A version is the commit SHA that last changed
the file.

The public Hub at [hub.calm.finos.org](https://hub.calm.finos.org/) is read-only and
accepts no external namespaces.

## Source

Generated from the fluxrig scenario file `payment_switch_two_regions.yaml`. The
scenario file is the same input the Rack runs. Wiring the generator into the
`fluxrig` build is planned.

## License

Apache-2.0.
