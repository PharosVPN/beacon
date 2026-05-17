# beacon — Build Brief (subagent)

**Read first, in order:** `docs/BUILD.md` → `docs/DESIGN.md` → this file.
This is a delegated subproject. If the design is silent on a contract you need,
stop and raise it — do not invent one.

---

## What you are building

`beacon` is the relay: a stateless, public-facing transparent gRPC proxy. It
lets clients (`caravel`) reach a `helm` controller that has no public IP and no
inbound ports. It holds no state and makes no policy decisions — `helm` does all
of that.

## Reuse — this is mostly a lift-and-rebrand

The relay machinery already exists in the private `sultix` project at
`/Users/khalefa/Projects/sultix.ai/sultix`, **owned by the same operator**. Lift
and adapt:

- `internal/mcproxy/` — transparent gRPC proxy: raw-byte codec, stream director,
  metadata sanitization (strip spoofable headers, inject the verified device
  fingerprint).
- `internal/mctunnel/` — reverse-tunnel transport: the controller dials *out*
  to the relay, the relay accepts; substreams multiplexed over the outer
  mTLS connection; reconnect with exponential backoff; keepalive.
- The embedded-relay (in-process, in-memory pipe) wiring.
- The device-CA mTLS verification helpers.

**Rebrand obligation (`docs/BUILD.md` §4):** strip *every* `sultix`, `mcproxy`,
`mctunnel`, `x-sultix-*` identifier — package paths, type names, metadata keys,
comments, filenames. The `beacon` repo must contain zero trace of `sultix`.
Re-license all lifted files to AGPL-3.0.

## Behaviour

- **Client side (public):** terminate client mTLS; the client leaf must chain to
  the Device CA. Read the cert fingerprint → inject as the one trusted metadata
  value. Strip everything else a client could spoof.
- **Controller side:** two modes —
  - *Embedded:* run in-process inside `helm` over an in-memory pipe.
  - *Remote:* accept the reverse tunnel that `helm` dials out; multiplex each
    client RPC as a substream back through it.
- **Forward, don't inspect.** `beacon` proxies opaque gRPC streams (unary,
  server-stream, client-stream, bidi). It must never need to parse profile
  payloads — and profile bundles are E2E-encrypted ciphertext anyway.

## Milestones

| # | Output |
|---|---|
| R1 | Skeleton; lift + rebrand the proxy + tunnel packages from `sultix` |
| R2 | Client-side mTLS termination + metadata sanitization |
| R3 | Embedded mode (in-process with `helm`) |
| R4 | Remote reverse-tunnel mode (`helm` dials out) + reconnect/keepalive |
| R5 | Relay enrollment (bootstrap token, Fleet-CA relay cert) |
| R6 | Static-binary packaging + deploy docs |

## Non-negotiables

- Stateless. No database. All lookups delegated to `helm`.
- `helm` dials `beacon`, never the reverse, in remote mode.
- `beacon` never sees plaintext profile bundles.
- Zero `sultix` lineage in the source.

## Depends on

The relay reverse-tunnel + relayed-client protos, owned by `helm`, in
`docs/proto/`. Coordinate any change with the `helm` build.
