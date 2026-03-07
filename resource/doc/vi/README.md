# webman là gì

webman là một framework dịch vụ hiệu suất cao được xây dựng trên Workerman, tích hợp HTTP, WebSocket, TCP, UDP và nhiều module khác. Thông qua các công nghệ như bộ nhớ thường trú, coroutine và connection pool, webman không chỉ vượt qua các hạn chế hiệu suất của PHP truyền thống mà còn mở rộng đáng kể các kịch bản ứng dụng.

Ngoài ra, webman cung cấp cơ chế plugin mạnh mẽ giúp nhà phát triển nhanh chóng tích hợp và tái sử dụng các module chức năng do nhà phát triển khác tạo ra. Dù là xây dựng website, phát triển API HTTP, nhắn tin tức thời, hệ thống IoT, game, dịch vụ TCP/UDP, dịch vụ Unix Socket hay khác, webman xử lý tất cả một cách dễ dàng với hiệu suất và tính linh hoạt xuất sắc.

# Triết lý của webman
**Cung cấp khả năng mở rộng tối đa và hiệu suất mạnh nhất với lõi tối thiểu.**

webman chỉ cung cấp các chức năng cốt lõi (định tuyến, middleware, phiên, giao diện quá trình tùy chỉnh). Các chức năng còn lại đều tái sử dụng từ hệ sinh thái Composer. Bạn có thể dùng các thành phần quen thuộc trong webman, chẳng hạn về cơ sở dữ liệu có thể chọn [illuminate/database](./db/tutorial.md) của Laravel, [ThinkORM](./db/thinkorm.md) của ThinkPHP hoặc thành phần khác như `Medoo`. Tích hợp chúng vào webman rất đơn giản.

# Đặc điểm của webman

1. Độ ổn định cao. webman được xây dựng trên workerman, một framework socket có độ ổn định cao và rất ít lỗi trong ngành.

2. Hiệu suất cực cao. webman có hiệu suất cao hơn framework php-fpm truyền thống khoảng 10–100 lần và khoảng gấp đôi các framework Go như gin, echo.

3. Khả năng tái sử dụng cao. Có thể tái sử dụng hệ sinh thái Composer hiện có mà không cần sửa đổi.

4. Khả năng mở rộng cao. Hỗ trợ quá trình tùy chỉnh, có thể làm mọi thứ mà workerman có thể làm.

5. Rất đơn giản, dễ dùng, chi phí học thấp, cách viết mã không khác framework truyền thống.

6. Hỗ trợ [đóng gói nhị phân](./others/bin.md), có thể chạy trực tiếp mà không cần môi trường PHP.

7. Sử dụng giấy phép MIT open source thoải mái và thân thiện với nhà phát triển nhất.

# Liên kết dự án
GitHub: https://github.com/walkor/webman **Đừng ngần ngại cho sao nhé!**

Gitee: https://gitee.com/walkor/webman **Đừng ngần ngại cho sao nhé!**

# Dữ liệu benchmark từ bên thứ ba

[![](../assets/img/benchmark1.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

Với các thao tác truy vấn cơ sở dữ liệu, webman đạt tới 390.000 QPS trên một máy duy nhất, cao gấp khoảng 80 lần so với framework Laravel trên kiến trúc php-fpm truyền thống.

[![](../assets/img/benchmarks-go.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

Với các thao tác truy vấn cơ sở dữ liệu, hiệu suất webman cao gấp khoảng hai lần so với framework web tương tự bằng ngôn ngữ Go.

Dữ liệu trên lấy từ [techempower.com](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf).
