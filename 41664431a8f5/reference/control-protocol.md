---
title: "Control protocol"
description: "The local TCP protocol of a running test build - transport, token, execution model, every op with its arguments and answers, and every error."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/reference/control-protocol/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/control-protocol.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/warm-copies.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/control-json.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/injection.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Control protocol

A test build started with `-scion-control` opens a control server: a line-based JSON protocol over TCP on the
loopback interface. The CLI uses it for warm copies and live injection; you can use it from any language. Requests
carry a token from `control.json`; the main thread executes them; the game never waits on the network.

## Transport

| Item | Rule |
|---|---|
| Address | Loopback interface only, on the port in `control.json` (`-scion-control 0` lets the OS choose). |
| Framing | One request per line: UTF-8 JSON, at most 64 KiB, ending in `\n`. One response line per request, in order. |
| Connections | Up to 8 clients; a connection may stay open for many requests. |
| Slow clients | Writes time out after 5 seconds and the connection is closed. |
| Availability | Builds with `SCION_SLOTS` or `SCION_TRIALS`. |

## Token

`control.json` and the live file hold a 128-bit random token as 32 hex digits. Every request includes it as `token`;
the server compares it in constant time. A wrong token gets `bad_token`.

## Execution model

The network thread only reads lines and queues them; the main thread executes them between frames. Idle, the server
allocates nothing per frame. While a queue runs, `inject` answers `pending: true` and applies at the next recipe
boundary; `run` answers when its queue ends.

## Requests and responses

Request:

<!-- validate: control-request -->
```json
{"id":7,"token":"9c1f0e7a2b3c4d5e6f708192a3b4c5d6","op":"inject","dll":"/Users/alex/orchard/.scion/grafts/ScionGrafts_51c2.dll","isolate":false}
```

Error response:

<!-- validate: control-response -->
```json
{"id":8,"ok":false,"error":{"code":"busy","message":"a queue is already running","hint":"wait for it or send quit"}}
```

## Ops

| op | Arguments | Answer (besides `id`, `ok`) | Without the define |
|---|---|---|---|
| `hello` | | `pid`, `port`, `build_id`, `abi`, `scion`, `role`, `out`, `state` | |
| `status` | | `state` (`idle`, `preparing`, `running`), `recipe`, `round`, `done`, `total`, `grafts`, `pending` | |
| `inject` | `dll`, `isolate` (false) | The `InjectReport` fields; `ok` is `InjectReport.Ok` | `not_supported` without `SCION_SLOTS` |
| `restore` | | `grafts` | `not_supported` without `SCION_SLOTS` |
| `disable` | `slot` | `found` | `not_supported` without `SCION_SLOTS` |
| `run` | `only` (null), `rounds` (1), `out` (required) | When the queue ends: `exit`, `out` | `not_supported` without `SCION_TRIALS` |
| `quit` | | `{}`, then the process exits with 0 | |

## Errors

| Code | Meaning |
|---|---|
| `bad_json` | The line is not JSON. |
| `line_too_long` | Over 64 KiB; the connection is closed. |
| `bad_token` | Wrong token. |
| `unknown_op` | Not one of the ops above. |
| `bad_args` | Missing or wrong-typed arguments. |
| `busy` | `run` while a queue is running. |
| `not_supported` | The op needs a define this build lacks. |
| `shutting_down` | After `quit`. |

## Example session

```sh
PORT=$(jq -r .port .scion/serve/q0/control.json); TOKEN=$(jq -r .token .scion/serve/q0/control.json)
printf '{"id":1,"token":"%s","op":"status"}\n' "$TOKEN" | nc localhost "$PORT"
```

```json
{"id":1,"ok":true,"state":"idle","recipe":null,"round":0,"done":0,"total":0,"grafts":3,"pending":false}
```

## See also

- [Warm copies](/scion/41664431a8f5/trials/warm-copies/)
- [control.json and live files](/scion/41664431a8f5/reference/files/control-json/)
- [Live injection](/scion/41664431a8f5/grafts/injection/)
- [Error and warning codes](/scion/41664431a8f5/reference/errors/)
