# beacon

> A signal relayed onward — so the lighthouse can stay hidden.

**`beacon` is the PharosVPN relay.** It is a stateless, public, mTLS-terminating
proxy that lets end-user clients reach a controller that has no public presence.
The controller (`helm`) stays behind NAT; `beacon` is the only public ingress
for clients.

Part of the [PharosVPN](https://github.com/PharosVPN) platform — see
[`docs/DESIGN.md`](https://github.com/PharosVPN/docs/blob/main/DESIGN.md).

## Role

- **Public ingress for clients only.** Terminates client mTLS, forwards their
  gRPC streams to `helm`.
- **Stateless.** No database. Every identity lookup is delegated to `helm`.
- **Sanitizing.** Strips spoofable client metadata; injects exactly one trusted
  value — the verified device fingerprint.
- **Two transports to `helm`:** *embedded* (in-process inside `helm`) or
  *remote reverse tunnel* (`helm` dials out to a public `beacon`). Identical
  trust either way.
- **Sees only ciphertext.** Profile bundles cross `beacon` end-to-end encrypted;
  a compromised remote `beacon` host cannot read user profiles.

## Stack

Go · transparent gRPC proxy · reverse-tunnel transport (multiplexed) · mTLS.

## Status

🚧 Pre-alpha — scaffolding. See [BUILD.md](BUILD.md).

## License

AGPL-3.0-or-later. Contributions under the DCO (`git commit -s`).
