# testing

`testing` provides reusable test doubles for Go standard-library network, HTTP,
TLS, and structured-logging interfaces.

## Installation

```bash
go get github.com/pbrpc/testing
```

## Packages

| Package                | Helpers |
| ---------------------- | ------- |
| `mocks/addr`           | Configurable `net.Addr` values |
| `mocks/certificate`    | Self-signed in-memory TLS certificates and PEM configuration values |
| `mocks/listener`       | Controllable, failing, and in-memory-pipe `net.Listener` implementations |
| `mocks/responsewriter` | An `http.ResponseWriter` that records headers and status while returning a body-write error |
| `mocks/roundtripper`   | Function adapters and request-recording `http.RoundTripper` implementations with response and failure helpers |
| `mocks/slog`           | An `slog.Handler` that captures log records |

## HTTP transport

`roundtripper.Record` records each request and its body, then answers through a
supplied function:

```go
wire := roundtripper.Record(
	roundtripper.Respond(http.StatusOK, nil, "ready"),
)
client := &http.Client{Transport: wire}

response, err := client.Get("http://service/health")
if err != nil {
	t.Fatal(err)
}
defer response.Body.Close()

sent := wire.Sent()
```

`roundtripper.Fail` supplies transport errors for connection-failure paths.

## Listeners

`listener.NewPipe` supplies a listener and the client side of its in-memory
connection:

```go
lis, clientConn := listener.NewPipe()
defer clientConn.Close()

served := make(chan error, 1)
go func() { served <- server.Serve(lis) }()
```

`listener.New` supplies a controllable listener, and `listener.NewFailing`
supplies an `Accept` error.

## TLS certificates

`certificate.SelfSigned` creates a `tls.Certificate`. `certificate.PEM` creates
the corresponding certificate and private-key strings used by configuration:

```go
cert := certificate.SelfSigned(t)
certPEM, keyPEM := certificate.PEM(t)
```

## HTTP responses and logs

`responsewriter.NewBroken` records response headers and status while returning
`responsewriter.ErrBroken` from body writes. `slog.NewCaptureHandler` appends
every handled `slog.Record` to a caller-owned slice.
