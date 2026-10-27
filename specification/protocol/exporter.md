<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
<!-- END doctoc generated TOC please keep comment here to allow auto update -->

<!--- Hugo front matter used to generate the website version of this page:
linkTitle: Exporter
--->

# OpenTelemetry Protocol Exporter

**Status**: [Stable](../document-status.md), except where otherwise specified

This document specifies the configuration options, authentication, and retry behavior available to the OpenTelemetry Protocol ([OTLP](../../oteps/0035-opentelemetry-protocol.md)) Exporter.

## Configuration Options

The following configuration options MUST be available to configure the OTLP exporter.
Each configuration option MUST be overridable by a signal specific option.

- **Endpoint (OTLP/HTTP)**: Target URL to which the exporter is going to send spans, metrics, or logs.
  The implementation MUST honor the following [URL components](https://datatracker.ietf.org/doc/html/rfc3986#section-3):
  - scheme (`http` or `https`)
  - host
  - port
  - path

  The implementation MAY ignore all other URL components.

  A scheme of `https` indicates a secure connection.
  When using `OTEL_EXPORTER_OTLP_ENDPOINT`, exporters MUST construct per-signal URLs as [described below](#endpoint-urls-for-otlphttp). The per-signal endpoint configuration options take precedence and can be used to override this behavior (the URL is used as-is for them, without any modifications). See the [OTLP Specification][otlphttp-req] for more details.
  - Default:  `http://localhost:4318` [1]
  - Env vars: `OTEL_EXPORTER_OTLP_ENDPOINT` `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT` `OTEL_EXPORTER_OTLP_METRICS_ENDPOINT` `OTEL_EXPORTER_OTLP_LOGS_ENDPOINT`
  - Type: [String][]

- **Endpoint (OTLP/gRPC)**: Target to which the exporter is going to send spans, metrics, or logs. The option SHOULD accept any form allowed by the underlying gRPC client implementation. Additionally, the option MUST accept a URL with a scheme of either `http` or `https`. A scheme of `https` indicates a secure connection and takes precedence over the `insecure` configuration setting. A scheme of `http` indicates an insecure connection and takes precedence over the `insecure` configuration setting. If the gRPC client implementation does not support an endpoint with a scheme of `http` or `https` then the endpoint SHOULD be transformed to the most sensible format for that implementation.
  - Default: `http://localhost:4317` [1]
  - Env vars: `OTEL_EXPORTER_OTLP_ENDPOINT` `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT` `OTEL_EXPORTER_OTLP_METRICS_ENDPOINT` `OTEL_EXPORTER_OTLP_LOGS_ENDPOINT`
  - Type: [String][]

- **Insecure**: Whether to enable client transport security for the exporter's gRPC connection. This option only applies to OTLP/gRPC when an endpoint is provided without the `http` or `https` scheme - OTLP/HTTP always uses the scheme provided for the `endpoint`. Implementations MAY choose to not implement the `insecure` option if it is not required or supported by the underlying gRPC client implementation.
  - Default: `false`
  - Env vars: `OTEL_EXPORTER_OTLP_INSECURE` `OTEL_EXPORTER_OTLP_TRACES_INSECURE` `OTEL_EXPORTER_OTLP_METRICS_INSECURE` `OTEL_EXPORTER_OTLP_LOGS_INSECURE` [2]
  - Type: [Boolean][]

- **Certificate File**: The trusted certificate to use when verifying a server's TLS credentials. Should only be used for a secure connection.
  - Default: n/a
  - Env vars: `OTEL_EXPORTER_OTLP_CERTIFICATE` `OTEL_EXPORTER_OTLP_TRACES_CERTIFICATE` `OTEL_EXPORTER_OTLP_METRICS_CERTIFICATE` `OTEL_EXPORTER_OTLP_LOGS_CERTIFICATE`
  - Type: [String][]

- **Client key file**: Clients private key to use in mTLS communication in PEM format.
  - Default: n/a
  - Env vars: `OTEL_EXPORTER_OTLP_CLIENT_KEY` `OTEL_EXPORTER_OTLP_TRACES_CLIENT_KEY` `OTEL_EXPORTER_OTLP_METRICS_CLIENT_KEY` `OTEL_EXPORTER_OTLP_LOGS_CLIENT_KEY`
  - Type: [String][]

- **Client certificate file**: Client certificate/chain trust for clients private key to use in mTLS communication in PEM format.
  - Default: n/a
  - Env vars: `OTEL_EXPORTER_OTLP_CLIENT_CERTIFICATE` `OTEL_EXPORTER_OTLP_TRACES_CLIENT_CERTIFICATE` `OTEL_EXPORTER_OTLP_METRICS_CLIENT_CERTIFICATE` `OTEL_EXPORTER_OTLP_LOGS_CLIENT_CERTIFICATE`
  - Type: [String][]

- **Headers**: Key-value pairs to be used as headers associated with gRPC or HTTP requests. See [Specifying headers](./exporter.md#specifying-headers-via-environment-variables) for more details.
  - Default: n/a
  - Env vars: `OTEL_EXPORTER_OTLP_HEADERS` `OTEL_EXPORTER_OTLP_TRACES_HEADERS` `OTEL_EXPORTER_OTLP_METRICS_HEADERS` `OTEL_EXPORTER_OTLP_LOGS_HEADERS`
  - Type: [String][]

- **Compression**: Compression key for supported compression types. Supported compression: `gzip`.
  - Default: No value [3]
  - Env vars: `OTEL_EXPORTER_OTLP_COMPRESSION` `OTEL_EXPORTER_OTLP_TRACES_COMPRESSION` `OTEL_EXPORTER_OTLP_METRICS_COMPRESSION` `OTEL_EXPORTER_OTLP_LOGS_COMPRESSION`
  - Type: [Enum][]

- **Timeout**: Maximum time the OTLP exporter will wait for each batch export,
  including all request attempts and waits between retries. The timeout starts
  when the batch export begins and MUST NOT be reset for each request attempt.
  This is an overall export timeout, distinct from connection or individual
  request attempt timeouts.
  - Default: 10s
  - Env vars: `OTEL_EXPORTER_OTLP_TIMEOUT` `OTEL_EXPORTER_OTLP_TRACES_TIMEOUT` `OTEL_EXPORTER_OTLP_METRICS_TIMEOUT` `OTEL_EXPORTER_OTLP_LOGS_TIMEOUT`
  - Type: [Timeout][]

- **Max Request Size**: Maximum size, in bytes, of a request message the exporter will send, as defined in the OTLP specification for [OTLP/gRPC][otlp-grpc-request] and [OTLP/HTTP][otlp-http-request].
  - Default: 67108864 (64 MiB = 64*1024*1024)
  - Type: [Integer][]

- **Max Response Size**: Maximum size, in bytes, of a response message the exporter will accept, as defined in the OTLP specification for [OTLP/gRPC][otlp-grpc-response] and [OTLP/HTTP][otlp-http-response].
  - Default: 4194304 (4 MiB = 4*1024*1024)
  - Type: [Integer][]

- **Protocol**: The transport protocol. Options MUST be one of: `grpc`, `http/protobuf`, `http/json`.
  See [Specify Protocol](./exporter.md#specify-protocol) for more details.
  - Default: `http/protobuf` [4]
  - Env vars: `OTEL_EXPORTER_OTLP_PROTOCOL` `OTEL_EXPORTER_OTLP_TRACES_PROTOCOL` `OTEL_EXPORTER_OTLP_METRICS_PROTOCOL` `OTEL_EXPORTER_OTLP_LOGS_PROTOCOL`
  - Type: [Enum][]

**[1]**: SDKs SHOULD default endpoint variables to use `http` scheme unless they have good reasons to choose
`https` scheme for the default (e.g., for backward compatibility reasons in a stable SDK release).

**[2]**: The environment variables `OTEL_EXPORTER_OTLP_SPAN_INSECURE`
and `OTEL_EXPORTER_OTLP_METRIC_INSECURE` are obsolete because they do not follow
the common naming scheme of the other environment variables. However, if they are already implemented,
they SHOULD continue to be supported as they were part of a stable release of the specification.

**[3]**: If no compression value is explicitly specified, SIGs can default to the value they deem
most useful among the supported options. This is especially important in the presence of technical constraints,
e.g. directly sending telemetry data from mobile devices to backend servers.

**[4]**: The default protocol SHOULD be `http/protobuf`, unless there are strong reasons for SDKs to select `grpc` as the default. For instance, maintaining backward compatibility may require keeping `grpc` as the default if it was previously established in a stable SDK release.

Supported values for `OTEL_EXPORTER_OTLP_*COMPRESSION` options:

- `none` if compression is disabled.
- `gzip` is the only specified compression method for now.

### Endpoint URLs for OTLP/HTTP

Based on the environment variables above, the OTLP/HTTP exporter MUST construct URLs
for each signal as follow:

1. For the per-signal variables (`OTEL_EXPORTER_OTLP_<signal>_ENDPOINT`), the URL
   MUST be used as-is without any modification. The only exception is that if an
   URL contains no path part, the root path `/` MUST be used (see [Example 2](#example-2)).
2. If signals are sent that have no per-signal configuration from the previous point,
   `OTEL_EXPORTER_OTLP_ENDPOINT` is used as a base URL and the signals are sent
   to these paths relative to that:

   * Traces: `v1/traces`
   * Metrics: `v1/metrics`.
   * Logs: `v1/logs`.

   Non-normatively, this could be implemented by ensuring that the base URL ends with
   a slash and then appending the relative URLs as strings.

An SDK MUST NOT modify the URL in ways other than specified above. That also means,
if the port is empty or not given, TCP port 80 is the default for the `http` scheme
and TCP port 443 is the default for the `https` scheme, as per the usual rules
for these schemes ([RFC 7230](https://datatracker.ietf.org/doc/html/rfc7230#section-2.7.1)).

#### Example 1

The following configuration sends all signals to the same collector:

```bash
export OTEL_EXPORTER_OTLP_ENDPOINT=http://collector:4318
```

Traces are sent to `http://collector:4318/v1/traces`, metrics to
`http://collector:4318/v1/metrics` and logs to `http://collector:4318/v1/logs`.

#### Example 2

Traces and metrics are sent to different collectors and paths:

```bash
export OTEL_EXPORTER_OTLP_TRACES_ENDPOINT=http://collector:4318
export OTEL_EXPORTER_OTLP_METRICS_ENDPOINT=https://collector.example.com/v1/metrics
```

This will send traces directly to the root path `http://collector:4318/`
(`/v1/traces` is only automatically added when using the non-signal-specific
environment variable) and metrics
to `https://collector.example.com/v1/metrics`, using the default https port (443).

#### Example 3

The following configuration sends all signals except for metrics to the same collector:

```bash
export OTEL_EXPORTER_OTLP_ENDPOINT=http://collector:4318/mycollector/
export OTEL_EXPORTER_OTLP_METRICS_ENDPOINT=https://collector.example.com/v1/metrics/
```

Traces are sent to `http://collector:4318/mycollector/v1/traces`,
logs to `http://collector:4318/mycollector/v1/logs`
and metrics to `https://collector.example.com/v1/metrics/`, using the default
https port (443).
Other signals, (if there were any) would be sent to their specific paths
relative to `http://collector:4318/mycollector/`.

### Specify Protocol

The `OTEL_EXPORTER_OTLP_PROTOCOL`, `OTEL_EXPORTER_OTLP_TRACES_PROTOCOL`, `OTEL_EXPORTER_OTLP_METRICS_PROTOCOL` and `OTEL_EXPORTER_OTLP_LOGS_PROTOCOL` environment variables specify the OTLP transport protocol. Supported values:

- `grpc` for protobuf-encoded data using gRPC wire format over HTTP/2 connection
- `http/protobuf` for protobuf-encoded data over HTTP connection
- `http/json` for JSON-encoded data over HTTP connection

SDKs SHOULD support both `grpc` and `http/protobuf` transports and MUST
support at least one of them. If they support only one, it SHOULD be
`http/protobuf`. They also MAY support `http/json`.

If no configuration is provided the default transport SHOULD be `http/protobuf`
unless SDKs have good reasons to choose `grpc` as the default (e.g. for backward
compatibility reasons when `grpc` was already the default in a stable SDK
release).

### Specifying headers via environment variables

The `OTEL_EXPORTER_OTLP_HEADERS`, `OTEL_EXPORTER_OTLP_TRACES_HEADERS`, `OTEL_EXPORTER_OTLP_METRICS_HEADERS`, `OTEL_EXPORTER_OTLP_LOGS_HEADERS` environment variables will contain a list of key value pairs, and these are expected to be represented in a format matching to the [W3C Baggage](https://www.w3.org/TR/baggage/#header-content) i.e., `key1=value1,key2=value2`. [Semi-colon delimited metadata](https://www.w3.org/TR/baggage/#property) is not supported. All attribute values MUST be considered strings.

## Authentication

**Status**: [Development](../document-status.md)

OTLP HTTP and gRPC exporters across traces, metrics, and logs SHOULD support
configuring an [`Authenticator`](../auth/sdk.md) programmatically and via
[Declarative Configuration](../configuration/sdk.md).

### Behavior with Header or Metadata Suppliers

When an OTLP exporter is configured with an `Authenticator` that supplies
dynamic headers or metadata to the exporter:

* **Per-Attempt Evaluation**: The exporter MUST invoke the `Authenticator` for
  each outgoing export request attempt (including retry attempts) rather than
  only at exporter initialization.
* **Header and Metadata Precedence**: The exporter MUST merge its configured
  headers with the dynamic headers or metadata returned by the `Authenticator`
  for each export attempt. If the same header or metadata key is defined in
  multiple places (compared case-insensitively), the `Authenticator`-supplied
  value MUST take precedence over any dynamic header callback (where supported
  by the SDK) and static `Headers` (configured programmatically or via
  `OTEL_EXPORTER_OTLP_*_HEADERS`) for that request, without mutating the
  exporter's static `Headers`.
* **gRPC Metadata Key Normalization**: For `grpc` exporters, the exporter MUST
  normalize header/metadata keys returned by the `Authenticator` to lowercase
  ASCII when constructing gRPC request metadata.
* **Shared Export Timeout Budget**: The exporter MUST pass the export
  operation's remaining timeout, deadline, or cancellation signal to the
  `Authenticator` (where the interface accepts one). Time spent executing the
  `Authenticator` MUST count against the export attempt's remaining timeout
  budget; if the timeout or deadline expires during authentication, the exporter
  MUST fail the export attempt immediately without transmitting the request over
  the network.

### Behavior with Transport or Credential Wrappers

When an OTLP exporter is configured with an `Authenticator` that wraps the HTTP
transport or provides gRPC per-RPC credentials:

* The exporter MUST still attach its configured static `Headers` to each
  outgoing request or RPC (for HTTP transport wrappers, before the wrapper
  executes so that wrapper-supplied headers override colliding static headers
  without mutating the exporter's static `Headers`).
* The exporter MUST wire the `Authenticator` onto its transport, channel, or
  RPC call options such that the exporter's configured TLS/mTLS (`Certificate File`,
  `Client key file`, `Client certificate file`), `Compression`, and `Timeout`
  settings are preserved rather than overwritten, and so that per-RPC
  credentials apply on each RPC attempt (including when a user-supplied gRPC
  connection is used).

### Telemetry Suppression, Diagnostics, and Lifecycle

When an OTLP exporter is configured with an `Authenticator`:

* **Suppressing Recursive Telemetry**: In SDKs that provide an in-band `Context`
  mechanism for suppressing instrumentation (see
  [Avoiding telemetry-induced-telemetry loops](../self-observability-supplementary-guidelines.md#avoiding-telemetry-induced-telemetry-loops)),
  the exporter SHOULD invoke the `Authenticator` within a
  suppressed-instrumentation context so that outbound network requests made
  during credential refresh do not trigger recursive telemetry export loops.
* **Error Handling**: If invoking the `Authenticator` fails or raises an error
  during an export attempt, the exporter MUST handle the error in accordance
  with [Error handling](../error-handling.md) rather than throwing an unhandled
  exception, and complete the export attempt with a failure result.
* **Diagnostic Redaction**: The exporter MUST NOT log dynamic header or metadata
  values returned by the `Authenticator`.
* **Shutdown**: The exporter MUST invoke the `Authenticator`'s `Shutdown`
  operation (if present) when the exporter's `Shutdown` is called.
* **Protocol Compatibility**: If an `Authenticator` only supports a subset of
  OTLP transport protocols (HTTP vs. gRPC), the exporter SHOULD fail
  initialization when configured with an incompatible `Authenticator`.

## Retry

Transient errors MUST be handled with a retry strategy. This retry strategy MUST implement an exponential back-off with jitter to avoid overwhelming the destination until the network is restored or the destination has recovered.

Each request attempt and wait between retries MUST be bounded by the remaining
time allowed by the exporter timeout. Any earlier deadline provided by the caller
(for example, a span processor, log processor, or metric reader) MUST take
precedence. When either limit is reached, the exporter MUST stop attempting to
export the batch, including any waits between retries, and complete the export
with a failure result.

### Transient errors

Transient errors are defined by the
[OTLP protocol specification][protocol-spec].

For [OTLP/gRPC][otlp-grpc], transient errors are defined by a set of
[retryable gRPC status codes][retryable-grpc-status-codes].

For [OTLP/HTTP][otlp-http], transient errors are defined by:

1. A set of [retryable HTTP status codes][retryable-http-status-codes] received
   from the server.
2. The scenarios described in:
   [All other responses](https://github.com/open-telemetry/opentelemetry-proto/blob/main/docs/specification.md#all-other-responses)
   and
   [OTLP/HTTP Connection](https://github.com/open-telemetry/opentelemetry-proto/blob/main/docs/specification.md#otlphttp-connection).

## User Agent

OpenTelemetry protocol exporters SHOULD emit a User-Agent header to at a minimum identify the exporter, the language of its implementation, and the version of the exporter. For example, the Python OTLP exporter version 1.2.3 would report the following:

```
OTel-OTLP-Exporter-Python/1.2.3
```

The format of the header SHOULD follow [RFC 7231][rfc-7231]. The conventions used for specifying the OpenTelemetry SDK language and version are available in the [Resource semantic conventions][resource-semconv].

Exporters MAY expose a configuration option to add a product identifier to the User-Agent header. The resulting User-Agent SHOULD include the exporter's default User-Agent string. The intention is to support an identifier for an OpenTelemetry SDK/Agent [distribution][opentelemetry-distribution]. Typically an exporter would *prepend* the given identifier to its own. For example:

```
MyDistribution/x.y.z OTel-OTLP-Exporter-Python/1.2.3
```

[Boolean]: ../configuration/sdk-environment-variables.md#boolean
[Timeout]: ../configuration/common.md#timeout
[String]: ../configuration/common.md#string
[Enum]: ../configuration/common.md#enum
[Integer]: ../configuration/common.md#integer

[resource-semconv]: https://github.com/open-telemetry/semantic-conventions/blob/main/docs/resource/README.md#telemetry-sdk
[otlphttp-req]: https://github.com/open-telemetry/opentelemetry-proto/blob/main/docs/specification.md#otlphttp-request
[rfc-7231]: https://datatracker.ietf.org/doc/html/rfc7231#section-5.5.3
[protocol-spec]: https://github.com/open-telemetry/opentelemetry-proto/blob/main/docs/specification.md
[otlp-grpc]: https://github.com/open-telemetry/opentelemetry-proto/blob/main/docs/specification.md#otlpgrpc
[otlp-http]: https://github.com/open-telemetry/opentelemetry-proto/blob/main/docs/specification.md#otlphttp
[otlp-grpc-request]: https://github.com/open-telemetry/opentelemetry-proto/blob/main/docs/specification.md#otlpgrpc-request
[otlp-grpc-response]: https://github.com/open-telemetry/opentelemetry-proto/blob/main/docs/specification.md#otlpgrpc-response
[otlp-http-request]: https://github.com/open-telemetry/opentelemetry-proto/blob/main/docs/specification.md#otlphttp-request
[otlp-http-response]: https://github.com/open-telemetry/opentelemetry-proto/blob/main/docs/specification.md#otlphttp-response
[retryable-grpc-status-codes]: https://github.com/open-telemetry/opentelemetry-proto/blob/main/docs/specification.md#failures
[retryable-http-status-codes]: https://github.com/open-telemetry/opentelemetry-proto/blob/main/docs/specification.md#failures-1
[opentelemetry-distribution]: https://opentelemetry.io/docs/concepts/distributions/
