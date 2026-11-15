> **English** | [Tiếng Việt](README.vi.md)

# t1k-go

Go implementation of the T1K protocol for [HiTechCloud vCWAF](https://github.com/hitechcloudvn/HiTechCloud_vCWAF) Web Application Firewall.

## Installation

```bash
go get github.com/hitechcloudvn/t1k-go
```

## Usage

Create a detector client and check every HTTP request against a vCWAF detector. Full programs live in [examples](./examples) (the health-check variant is built with `go build -tags healthcheck ./examples`).

```go
package main

import (
	"fmt"
	"net/http"
	"os"
	"time"

	t1k "github.com/hitechcloudvn/t1k-go"
)

func main() {
	server, err := t1k.NewWithPoolSizeWithTimeout(os.Getenv("DETECTOR_ADDR"), 10, 10*time.Second)
	if err != nil {
		panic(err)
	}
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		res, err := server.DetectHttpRequest(r)
		if err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)
			return
		}
		if res.Blocked() {
			http.Error(w, fmt.Sprintf("blocked event id %s", res.EventID()), res.StatusCode())
			return
		}
		_, _ = w.Write([]byte("allowed"))
	})
	_ = http.ListenAndServe(":80", nil)
}
```

## Health check

Set a `HealthCheckConfig` to probe detector endpoints over the HTTP or T1K protocol (see [examples/enable_health_check.go](./examples/enable_health_check.go)).

## License

See [License](./License).
