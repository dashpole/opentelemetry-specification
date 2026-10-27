<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Overview](#overview)
- [Package and Interface Scope](#package-and-interface-scope)
- [Capabilities and Constraints](#capabilities-and-constraints)
  - [Capabilities](#capabilities)
  - [Constraints](#constraints)
- [Documentation Requirements](#documentation-requirements)

<!-- END doctoc -->

</details>

## Overview

An `Authenticator` is an SDK extension component responsible for acquiring,
caching, refreshing, and attaching dynamic client credentials to outgoing
telemetry export requests (such as OAuth 2.0 bearer tokens, cloud provider
ambient credentials, or rotating API tokens).

While the [OpenTelemetry Protocol (OTLP) Exporter](../protocol/exporter.md) is
the primary built-in consumer of `Authenticator`, `Authenticator` is defined as
a general exporter authentication concept for HTTP and gRPC transports rather
than coupled exclusively to OTLP.

## Package and Interface Scope

SDKs SHOULD define the `Authenticator` interface in a common SDK or shared
exporter-transport package rather than inside an OTLP-specific exporter package
so that:

1. Contrib and third-party authentication plugins do not need to depend on the
   OTLP exporter package.
2. Non-OTLP HTTP or gRPC exporters can reuse the same authentication plugins.
3. Separate per-signal or per-protocol exporter modules share a common contract.

Because programming languages have different HTTP and gRPC transport stacks and
concurrency models, the specification does not mandate a single low-level
method signature across languages. Instead, each language SDK defines the
idiomatic interface (or HTTP and gRPC interfaces) that represents an
`Authenticator` in that language—for example:

* Language-native HTTP transport wrappers and gRPC per-RPC credential
  interfaces (such as wrapping an `http.RoundTripper` and providing gRPC
  `PerRPCCredentials` / `CallCredentials`).
* A per-request header or metadata supplier (such as a synchronous or
  asynchronous function returning key-value header pairs).

## Capabilities and Constraints

### Capabilities

An SDK's `Authenticator` mechanism SHOULD support:

* **Per-Request Dynamic Headers or Transport Credentials**: Dynamically
  providing one or more HTTP headers or gRPC metadata entries per outgoing
  request attempt (for example, `Authorization` and tenant/routing headers), or
  wrapping/configuring the underlying HTTP transport or gRPC per-RPC
  credentials where idiomatic.
* **Request Context and Timeout Propagation**: Accepting or accessing request
  context—such as the target endpoint URI/authority and a
  timeout, deadline, or cancellation signal—when needed to scope credentials or
  bound out-of-band token refresh calls.
* **Shutdown (`Shutdown`)**: An optional idempotent lifecycle operation to
  release background resources (such as token-exchange HTTP clients or file
  watchers) when the owning exporter shuts down.

### Constraints

An SDK's `Authenticator` mechanism:

* MUST NOT require the exporter to materialize or pass the full serialized
  request payload (`byte[]`) in memory, preserving streaming serialization.
* MUST NOT require access to in-memory telemetry items (spans, metrics, or log
  records), keeping authentication strictly at the transport layer.

## Documentation Requirements

SDKs MUST document—either on the `Authenticator` interface or on the exporter
configuration option that accepts an `Authenticator`—the following expectations
for `Authenticator` implementations:

1. **Concurrency safety**: Safe for concurrent invocation across multiple
   threads or coroutines.
2. **Timeout and cancellation adherence**: Honoring any cancellation, deadline,
   or timeout signal provided by the caller when performing out-of-band
   credential acquisition.
3. **Credential redaction**: Never logging sensitive credential values or
   dynamic authentication header values in diagnostic output.
