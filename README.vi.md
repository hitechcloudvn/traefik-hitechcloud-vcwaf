> [English](README.md) | **Tiếng Việt**

# Plugin Traefik HiTechCloud vCWAF

Plugin này là middleware cho Traefik, dùng để phát hiện và chặn các request độc hại dựa trên engine [HiTechCloud vCWAF](https://hitechwaf.vn/).

## Chuẩn bị HiTechCloud vCWAF
Engine phát hiện của HiTechCloud vCWAF mặc định cung cấp dịch vụ qua Unix socket. Ta cần chuyển sang TCP để plugin t1k có thể gọi được.

1. Vào thư mục cấu hình của engine phát hiện HiTechCloud vCWAF:
```shell
cd /data/hitechcloud-vcwaf/resources/detector/
```
2. Mở tệp `detector.yml` trong trình soạn thảo. Sửa cấu hình bind từ Unix socket sang TCP bằng cách thêm các thiết lập sau:
```yaml
bind_addr: 0.0.0.0
listen_port: 8000
```
Các giá trị cấu hình này sẽ ghi đè thiết lập mặc định trong container, khiến engine HiTechCloud vCWAF lắng nghe trên cổng 8000.

3. Tiếp theo, map cổng 8000 của container ra máy host. Trước tiên, vào thư mục cài đặt HiTechCloud vCWAF:
```shell
cd /data/hitechcloud-vcwaf
```

4. Mở tệp compose.yaml trong trình soạn thảo và thêm trường ports vào container detector để mở cổng 8000:
```yaml
...
detect:
  ports:
    - 8000:8000
...
```

5. Lưu thay đổi và khởi động lại HiTechCloud vCWAF bằng các lệnh sau:
```shell
docker-compose down
docker-compose up -d
```
Điều này sẽ áp dụng thay đổi và kích hoạt cấu hình mới.

## Cách dùng plugin

Để plugin có hiệu lực trên một instance Traefik, nó phải được khai báo trong cấu hình tĩnh (static configuration).

Plugin được phân tích và nạp độc quyền trong lúc khởi động, giúp Traefik kiểm tra tính toàn vẹn của mã và phát hiện lỗi sớm.
Nếu có lỗi khi nạp, plugin sẽ bị vô hiệu hoá.

Vì lý do bảo mật, không thể khởi động plugin mới hay sửa plugin hiện có trong khi Traefik đang chạy.

Sau khi được nạp, các middleware plugin hoạt động giống hệt các middleware biên dịch tĩnh.
Việc khởi tạo và hành vi của chúng được điều khiển bởi cấu hình động (dynamic configuration).

Các phụ thuộc của plugin phải được [vendored](https://golang.org/ref/mod#vendoring) cho từng plugin.
Các gói vendored nên được đưa vào kho GitHub của plugin. ([Go modules](https://blog.golang.org/using-go-modules) không được hỗ trợ.)

### Cấu hình

Với mỗi plugin, cấu hình tĩnh của Traefik phải khai báo tên module (như với các gói Go thông thường).

Khai báo sau (dạng YAML) định nghĩa một plugin:

```yaml
# Cấu hình tĩnh / Static configuration

experimental:
  plugins:
    hitechcloud-vcwaf:
      moduleName: github.com/hitechcloudvn/traefik-hitechcloud-vcwaf
      version: v1.0.0
```

Dưới đây là ví dụ cấu hình động của file provider (dạng YAML), trong đó phần quan trọng là mục `http.middlewares`:

```yaml
# Cấu hình động / Dynamic configuration

http:
  routers:
    my-router:
      rule: host(`demo.localhost`)
      service: service-foo
      entryPoints:
        - web
      middlewares:
        - hitechcloud-vcwaf

  services:
   service-foo:
      loadBalancer:
        servers:
          - url: http://127.0.0.1:5000

  middlewares:
    hitechcloud-vcwaf:
      plugin:
        hitechcloud-vcwaf:
          addr: hitechcloud-vcwaf-detector.hitechcloud-vcwaf:8000 # địa chỉ engine phát hiện HiTechCloud vCWAF
```
