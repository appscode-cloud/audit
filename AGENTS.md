# AGENTS.md

This file provides guidance to coding agents (e.g. Claude Code, claude.ai/code) when working with code in this repository.

## Repository purpose

Go module `go.bytebuilders.dev/audit` — a small library that publishes Kubernetes resource **audit events** (and derived billing events) to NATS so downstream services in the ByteBuilders/AppsCode platform can react: billing aggregation, search indexing, etc. Imported by other AppsCode controllers; no binary of its own.

## Architecture

- `api/v1/` — audit/billing event types and OpenAPI definitions.
- `lib/`:
  - `publisher.go` — public `Publisher` interface and constructor.
  - `nats.go` — NATS transport.
  - `lister.go` — discovery helpers (walk the cluster's resources).
  - `audit_event.go`, `billing_event.go` — event payload constructors.
  - `node.go`, `resource.go` — per-kind extractors that decide what to emit when something changes.
- `hack/`, `Makefile` — AppsCode build harness (codegen, fmt, lint).
- `vendor/` — checked-in deps.

This is a **library** — there's no binary, no Docker image, no install chart.

## Common commands

- `make ci` — full CI pipeline.
- `make gen` — regenerate API types / openapi from `api/v1/`.
- `make fmt`, `make lint`, `make unit-tests` / `make test` — standard.
- `make verify` — codegen + module-tidy verification.
- `make add-license` / `make check-license` — manage license headers.

## Conventions

- Module path is `go.bytebuilders.dev/audit` (vanity URL); imports must use that.
- License: see `LICENSE`. Sign off commits (`git commit -s`); contributions follow the DCO (`DCO`, `CONTRIBUTING.md`).
- Vendor directory is checked in; keep `go mod tidy && go mod vendor` clean.
- This is a **library**; every exported symbol in `lib/` and `api/v1/` is API. Don't break callers (KubeDB, KubeVault, ACE backend) without coordinated updates.
- NATS is the transport. Adding another backend means a new file under `lib/` (`kafka.go`, etc.) implementing the publisher interface; don't add a transport switch inside `publisher.go`.
- Per-kind extraction lives in `lib/resource.go` / `lib/node.go` — keep that pattern when adding new resource kinds (don't sprinkle decoder logic into `publisher.go`).
