<p align="center"><img src="https://raw.githubusercontent.com/go-coord/brand/main/social/go-coord.png" alt="go-coord" width="640"></p>

<h1 align="center">go-coord</h1>
<p align="center"><strong>Pure-Go cross-host coordination on etcd v3 — host liveness, watcher, and leader election. No cgo.</strong></p>

<p align="center">
  🌐 <a href="https://go-coord.github.io">Website</a> ·
  📚 <a href="https://go-coord.github.io/docs/">Documentation</a>
</p>

<p align="center">
  <a href="https://go-coord.github.io/docs/"><img alt="Docs" src="https://img.shields.io/badge/docs-mkdocs--material-0079A8?style=flat-square"></a>
  <a href="https://github.com/go-coord/coord/blob/main/LICENSE"><img alt="License: BSD-3-Clause" src="https://img.shields.io/badge/license-BSD--3--Clause-blue?style=flat-square"></a>
  <img alt="Go 1.26.4+" src="https://img.shields.io/badge/go-1.26.4%2B-00ADD8?style=flat-square&logo=go&logoColor=white">
  <img alt="Coverage 100%" src="https://img.shields.io/badge/coverage-100%25-1a7f37?style=flat-square">
</p>

---

go-coord is a **pure-Go (no cgo)** cross-host coordination layer built on
[etcd](https://etcd.io) v3. It gives a fleet of agents the primitives they need
to coordinate high-availability behaviour across hosts, without vendoring or a C
toolchain.

## Repositories

| Repo | What it is |
|------|------------|
| [**coord**](https://github.com/go-coord/coord) | the library: `HostLiveness` (TTL leases + self-healing keep-alive), `HostWatcher` (up/down events), `Election` / `ElectionPool` (leader election) |
| [**docs**](https://github.com/go-coord/docs) | MkDocs Material documentation, versioned with [mike], served at [/docs/](https://go-coord.github.io/docs/) |
| [**go-coord.github.io**](https://github.com/go-coord/go-coord.github.io) | the Hugo landing page |
| [**brand**](https://github.com/go-coord/brand) | logos and brand assets |

## The primitives

- **HostLiveness** registers a short-TTL lease at `/coord/hosts/<host_uuid>` and
  refreshes it in the background. Lease expiry is the cluster-wide signal that a
  host is down. The keep-alive loop **self-heals** if etcd drops the renewal
  stream, so a transient hiccup doesn't silently retire a healthy host forever.
- **HostWatcher** emits `HostEvent {Up|Down}` on a channel; a fresh watcher
  replays every existing host as a synthetic `Up` first, and `Down` events carry
  the host's last-seen metadata.
- **Election / ElectionPool** wrap etcd-concurrency leader election scoped to a
  key, so cross-host work coalesces to one leader per key. The pool keeps one
  long-lived session per key to avoid a grant/revoke cycle on every event.

## Principles

- **Pure Go, `CGO_ENABLED=0`** — trivial cross-compilation, a single static
  binary, no C toolchain.
- **No vendoring.** The etcd client is a normal, build-from-source module
  dependency.
- **Caller owns the connection.** The package takes an open `*clientv3.Client`;
  it never fans out extra connections.
- **100% test coverage** including every error branch, enforced as a CI gate,
  green across the six 64-bit Go targets (amd64, arm64, riscv64, loong64,
  ppc64le, s390x).

BSD-3-Clause.

[mike]: https://github.com/jimporter/mike
