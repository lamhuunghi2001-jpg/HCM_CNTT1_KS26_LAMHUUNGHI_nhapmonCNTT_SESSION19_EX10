

## Phần 1: Phân tích 2 giải pháp xác thực

### Câu 1. Ba yếu tố xác thực

Có 3 loại yếu tố để chứng minh mình là ai:

Something You Know (cái bạn biết): thứ nằm trong đầu, như mật khẩu, mã PIN.
Something You Have (cái bạn có): thứ cầm trên tay, như điện thoại, thẻ, khóa bảo mật USB.
Something You Are (cái bạn là): đặc điểm cơ thể, như vân tay, khuôn mặt, mống mắt.

Khi bật 2FA, mật khẩu là yếu tố Something You Know. Mã OTP là yếu tố Something You Have, vì mã chỉ có khi người dùng đang cầm điện thoại có cài app Authenticator. Hai yếu tố thuộc hai loại khác nhau nên kẻ gian biết mật khẩu vẫn chưa đăng nhập được.

### Câu 2. Nguyên lý của thuật toán TOTP

Khi cài 2FA, máy chủ và app Authenticator được chia sẻ chung một khóa bí mật (Shared Secret), thường qua việc quét mã QR một lần duy nhất. Sau đó cả hai bên tự tính mã theo cách sau:

1. Lấy thời gian hiện tại, chia thành các khoảng 30 giây. Mỗi khoảng là một số đếm.
2. Trộn khóa bí mật với số đếm đó bằng hàm băm HMAC.
3. Cắt kết quả lấy 6 chữ số. Đó là mã OTP.

Máy chủ cũng tính y hệt, nên hai mã giống nhau và máy chủ chỉ việc so sánh.

Vì sao chạy được khi bật chế độ máy bay: việc tính mã chỉ cần 2 thứ, là khóa bí mật đã lưu sẵn trong máy và đồng hồ của điện thoại. Cả hai đều có sẵn trong máy, không cần Wi-Fi, 4G hay SMS. Mã đổi mỗi 30 giây vì số đếm thời gian thay đổi sau mỗi 30 giây.

## Phần 2: So sánh Trade-off và ra quyết định

### Câu 3. So sánh SMS OTP và TOTP Authenticator

Tiêu chí 1: Chống SIM Swapping và nghe lén SS7
SMS OTP: kém. Kẻ gian có thể lừa nhà mạng cấp SIM mới (SIM Swapping) hoặc nghe lén sóng mạng (SS7) để lấy mã.
TOTP: rất tốt. Khóa bí mật nằm trong điện thoại, mã không truyền qua mạng viễn thông nên không bị chặn giữa đường.

Tiêu chí 2: Khi offline hoặc ở nước ngoài
SMS OTP: phụ thuộc sóng di động. Mất sóng hoặc ra nước ngoài mà không có roaming thì không nhận được mã.
TOTP: hoạt động offline hoàn toàn, kể cả trên máy bay.

Tiêu chí 3: Độ tiện dụng và chi phí
SMS OTP: quen thuộc, không cần cài thêm app, nhưng công ty phải trả phí gửi SMS cho mỗi lần.
TOTP: miễn phí, mở app là có mã. Chỉ cần cài app và quét QR một lần.

Tiêu chí 4: Rủi ro khi mất điện thoại
SMS OTP: làm lại SIM là lấy lại được số, nhưng chính điều này cũng giúp kẻ gian trộm SIM dễ dàng.
TOTP: mất máy là mất app, nên phải có Mã Khôi phục (Backup Codes) hoặc tính năng đồng bộ đám mây có mã hóa để lấy lại quyền truy cập.

### Câu 4. Kết luận lựa chọn

Ban Giám đốc nên chọn giải pháp 2, TOTP qua app Authenticator, cho toàn bộ đội kỹ sư.

Lý do thứ nhất là việc di chuyển. Nhiều lập trình viên đi công tác nước ngoài dự GDC hoặc ngồi trên chuyến bay không có sóng. SMS sẽ không đến được trong những lúc đó, còn TOTP vẫn sinh mã bình thường.

Lý do thứ hai là rủi ro SIM Swapping. Kẻ gian có thể dùng thủ thuật xã hội để lừa nhà mạng và chiếm số điện thoại của nhà sáng lập. Nếu dùng SMS, chúng nhận luôn mã OTP và vào được kho mã nguồn GitHub Enterprise cùng tài khoản thanh toán. TOTP không dính đến số điện thoại nên cách tấn công này vô hiệu.

Ngoài ra TOTP không tốn phí SMS. Nhược điểm duy nhất là mất điện thoại, và nhược điểm này được xử lý bằng Backup Codes ở phần sau.

## Phần 3: Quản lý rủi ro và Backup Codes

### Câu 5. Bộ 10 Mã Khôi phục

Mục đích: đây là chìa khóa dự phòng. Khi mất điện thoại, hỏng máy hoặc lỡ xóa app Authenticator, nhân viên dùng một mã khôi phục để đăng nhập và cài lại 2FA. Mỗi mã chỉ dùng được một lần.

Hai vị trí lưu trữ an toàn:
1. In ra giấy rồi cất trong két sắt hoặc ngăn tủ có khóa (ở công ty hoặc ở nhà).
2. Lưu trong két sắt số của Password Manager (như Bitwarden), nơi dữ liệu được mã hóa bằng Master Password.

Một sai lầm tai hại cần tránh: lưu mã dưới dạng ảnh chụp màn hình hoặc file ghi chú ngay trên chính chiếc điện thoại đang dùng Authenticator, hoặc gửi qua chat, email. Mất điện thoại là mất luôn cả app lẫn mã dự phòng, còn gửi qua chat hay email thì ai vào được tài khoản đó cũng đọc được.