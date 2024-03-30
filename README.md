# docker-compose-laravel

## Cách sử dụng

Để bắt đầu, hãy đảm bảo bạn đã [Cài đặt Docker](https://docs.docker.com/docker-for-mac/install/) trên hệ thống của mình, sau đó sao chép kho lưu trữ này.

Tiếp theo, điều hướng trong thiết bị đầu cuối của bạn đến thư mục mà bạn đã sao chép thư mục này và quay các vùng chứa cho máy chủ web bằng cách chạy `docker-compose up -d --build app`.

Sau khi hoàn tất, hãy làm theo các bước từ tệp [src/README.md](src/README.md) để thêm dự án Laravel của bạn vào (hoặc tạo một dự án trống mới).

**Lưu ý**: Tên máy chủ cơ sở dữ liệu MySQL của bạn phải là `mysql`, **không phải** `localhost`. Tên người dùng và cơ sở dữ liệu đều phải là `homestead` với mật khẩu `secret`.

Đưa mạng Docker Compose lên với `app` thay vì chỉ sử dụng `up`, đảm bảo rằng chỉ các vùng chứa của trang web của chúng tôi mới được hiển thị khi bắt đầu, thay vì tất cả các vùng chứa lệnh. Phần sau đây được xây dựng cho máy chủ web của chúng tôi, với các cổng được hiển thị chi tiết:

- **nginx** - `:80`
- **mysql** - `:3306`
- **php** - `:9000`
- **redis** - `:6379`
- **mailhog** - `:8025`

Ba vùng chứa bổ sung được bao gồm để xử lý các lệnh Composer, NPM và Artisan *mà không* cần phải cài đặt các nền tảng này trên máy tính cục bộ của bạn. Sử dụng các ví dụ lệnh sau từ thư mục gốc của dự án, sửa đổi chúng để phù hợp với trường hợp sử dụng cụ thể của bạn.

- `docker-compose run --rm composer update`
- `docker-compose run --rm npm run dev`
- `docker-compose run --rm artisan migrate`

## Vấn đề về quyền

Nếu bạn gặp bất kỳ sự cố nào với quyền hệ thống tệp khi truy cập ứng dụng của mình hoặc chạy lệnh vùng chứa, hãy thử hoàn thành một trong các nhóm bước bên dưới.

**Nếu bạn đang sử dụng máy chủ hoặc môi trường cục bộ của mình với tư cách là người dùng root:**

- Mang bất kỳ container nào xuống bằng `docker-compose down`
- Thay thế bất kỳ phiên bản nào của `php.dockerfile` trong tệp docker-compose.yml bằng `php.root.dockerfile`
- Xây dựng lại các container bằng cách chạy `docker-compose build --no-cache`

**Nếu bạn đang sử dụng máy chủ hoặc môi trường cục bộ của mình với tư cách là người dùng không phải root:**

- Mang bất kỳ container nào xuống bằng `docker-compose down`
- Trong thiết bị đầu cuối của bạn, hãy chạy `export UID=$(id -u)` và sau đó `export GID=$(id -g)`
- Nếu bạn thấy bất kỳ lỗi nào về biến chỉ đọc ở bước trên, bạn có thể bỏ qua và tiếp tục
- Xây dựng lại các container bằng cách chạy `docker-compose build --no-cache`

Sau đó, hãy khôi phục mạng vùng chứa của bạn hoặc chạy lại lệnh bạn đã thử trước đó và xem liệu cách đó có khắc phục được không.

## Lưu trữ MySQL liên tục

Theo mặc định, bất cứ khi nào bạn gỡ mạng Docker xuống, dữ liệu MySQL của bạn sẽ bị xóa sau khi các vùng chứa bị phá hủy. Nếu bạn muốn có dữ liệu liên tục còn sót lại sau khi hạ vùng chứa xuống và sao lưu, hãy làm như sau:

1. Tạo thư mục `mysql` trong thư mục gốc của dự án, cùng với các thư mục `nginx` và `src`.
2. Trong dịch vụ mysql trong tệp `docker-compose.yml` của bạn, hãy thêm các dòng sau:

```
khối lượng:
   - ./mysql:/var/lib/mysql
```

## Cách sử dụng trong sản xuất

Mặc dù ban đầu tôi tạo mẫu này để phát triển cục bộ nhưng nó đủ mạnh để sử dụng trong triển khai ứng dụng Laravel cơ bản. Khuyến nghị lớn nhất là đảm bảo rằng HTTPS được bật bằng cách bổ sung vào tệp `nginx/default.conf` và sử dụng một cái gì đó như [Let's Encrypt](https://hub.docker.com/r/linuxserver/letsencrypt) để tạo chứng chỉ SSL.

## Biên dịch tài sản

Cấu hình này có thể biên dịch nội dung bằng cả [laravel mix](https://laravel-mix.com/) và [vite](https://vitejs.dev/). Để bắt đầu, trước tiên bạn cần thêm ` --host 0.0.0.0` sau khi kết thúc lệnh dev có liên quan trong `package.json`. Vì vậy, ví dụ, với một dự án Laravel sử dụng Vite, bạn sẽ thấy:

```json
"kịch bản": {
   "dev": "vite --host 0.0.0.0",
   "build": "vite build"
},
```

Sau đó, chạy các lệnh sau để cài đặt các phần phụ thuộc của bạn và khởi động máy chủ nhà phát triển:

- `docker-compose run --rm npm install`
- `docker-compose run --rm --service-ports npm run dev`

Sau đó, bạn sẽ có thể sử dụng lệnh `@vite` để cho phép tải lại mô-đun nóng trên ứng dụng Laravel cục bộ của mình.

Bạn muốn xây dựng để sản xuất? Đơn giản chỉ cần chạy `docker-compose run --rm npm run build`.

## MailHog

Phiên bản hiện tại của Laravel (9 tính đến ngày hôm nay) sử dụng MailHog làm ứng dụng mặc định để kiểm tra việc gửi email và hoạt động chung của SMTP trong quá trình phát triển cục bộ. Bằng cách sử dụng hình ảnh Docker Hub được cung cấp, việc thiết lập và sẵn sàng một phiên bản thật đơn giản và dễ dàng. Dịch vụ này được bao gồm trong tệp `docker-compose.yml` và hoạt động cùng với các dịch vụ cơ sở dữ liệu và máy chủ web.

Để xem bảng điều khiển và xem mọi email đến qua hệ thống, hãy truy cập [localhost:8025](http://localhost:8025) sau khi chạy `docker-compose up -d
