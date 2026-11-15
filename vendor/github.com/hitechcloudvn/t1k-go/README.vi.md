> [English](README.md) | **Tiếng Việt**

# t1k-go

Triển khai giao thức T1K bằng Go cho Tường lửa ứng dụng web [HiTechCloud vCWAF](https://github.com/hitechcloudvn/HiTechCloud_vCWAF).

## Cài đặt

```bash
go get github.com/hitechcloudvn/t1k-go
```

## Sử dụng

Tạo một client detector và kiểm tra từng HTTP request với detector vCWAF. Các chương trình đầy đủ nằm trong [examples](./examples) (biến thể health-check được build với `go build -tags healthcheck ./examples`).

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

## Kiểm tra sức khỏe (Health check)

Cấu hình `HealthCheckConfig` để thăm dò các detector theo giao thức HTTP hoặc T1K (xem [examples/enable_health_check.go](./examples/enable_health_check.go)).

## Giấy phép

Xem [License](./License).
