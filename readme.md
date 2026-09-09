# OBS Camera Watch

Trang web đơn (1 file `index.html`, không cần cài đặt, không cần server) giúp **giám sát một camera/source trong OBS**, tự phát hiện khi hình bị "đơ" (đứng hình) và **tự động khởi động lại (deactivate → activate) đúng source đó** — thay vì chỉ ẩn/hiện overlay như cách làm thông thường.

Toàn bộ cấu hình (IP, port, password, tên source...) chỉ lưu trong `localStorage` của trình duyệt đang mở trang — **không gửi đi đâu khác**.

---

## Tính năng chính

- Kết nối tới OBS qua **OBS WebSocket v5** (obs-websocket-js, tải qua CDN có nhiều đường dự phòng).
- Theo dõi định kỳ một **source** trong OBS bằng cách chụp ảnh nhỏ (`GetSourceScreenshot`) và so sánh với lần trước để phát hiện "đơ" (hình không đổi trong X giây).
- Khi phát hiện lỗi (đơ, mất ảnh, source không lấy được ảnh), tự động **reload đúng cách theo từng loại source**, không phải kiểu ẩn/hiện overlay:
  - **Video Capture Device** (webcam — `dshow_input`/Windows, `av_capture_input`/macOS, `v4l2_input`/Linux): tháo rồi gán lại đúng ID thiết bị → buộc OBS ngắt & kết nối lại camera thật sự.
  - **Media/VLC Source**: gọi lệnh `TriggerMediaInputAction` để restart phát lại.
  - **Browser Source**: bấm nút "Refresh no cache" để tải lại trang.
  - Loại source khác: fallback ghi lại settings hiện có để buộc OBS nạp lại nguồn.
- Có **cooldown** giữa các lần reload để tránh reload liên tục.
- Log hoạt động ngay trên trang (kết nối, kiểm tra, reload, lỗi...).
- Tự động kết nối lại nếu OBS WebSocket bị rớt kết nối.
- Popup cài đặt kết nối có kèm **video hướng dẫn** bật OBS WebSocket, tự mở khi chưa có cấu hình.

---

## Yêu cầu

- OBS Studio 28 trở lên (đã tích hợp sẵn OBS WebSocket v5).
- Trong OBS: **Tools → WebSocket Server Settings** → bật **Enable WebSocket server**, ghi lại **Port** (mặc định `4455`) và **Password** (nếu có đặt).
- Trình duyệt trên máy/thiết bị mở file `index.html` phải có thể kết nối mạng tới máy đang chạy OBS (cùng mạng LAN, hoặc đã mở port ra ngoài nếu truy cập từ xa).

---

## Cách dùng

1. Mở file `index.html` bằng trình duyệt (double-click là chạy được, không cần server).
2. Ở lần mở đầu tiên, popup **"Cài đặt kết nối"** sẽ tự hiện ra. Xem video hướng dẫn trong popup nếu cần, sau đó điền:
   - **Địa chỉ IP OBS**: IP máy đang chạy OBS (dùng `localhost` nếu mở cùng máy).
   - **Port**: theo cấu hình WebSocket Server trong OBS (mặc định `4455`).
   - **Password**: để trống nếu không đặt password.
   - **Tên Source / Overlay camera**: đúng tên source (Input) trong OBS, phân biệt hoa/thường.
   - **Thông số giám sát** (có thể để mặc định):
     - Chu kỳ kiểm tra: mặc định `5` giây.
     - Timeout phát hiện đơ: mặc định `12` giây.
     - Cooldown reload: mặc định `20` giây.
3. Bấm **"Kết nối OBS"** — popup sẽ tự đóng khi kết nối thành công.
4. Theo dõi trạng thái camera ở khu vực **"Trạng thái"** và log hoạt động bên dưới.
5. Có thể bấm **"Kiểm tra ngay"** hoặc **"Reload Camera"** để thao tác thủ công bất cứ lúc nào.
6. Bấm **"Cài đặt kết nối"** trên thanh tóm tắt để mở lại popup và sửa thông số. Bấm **"Ngắt kết nối"** để dừng giám sát.

---

## Ghi chú quan trọng

- File này **không phải là extension hay app cài đặt** — chỉ là một trang HTML tĩnh, mở trực tiếp bằng trình duyệt.
- Cơ chế phát hiện "đơ" dựa trên so sánh ảnh chụp theo thời gian — là **tín hiệu suy luận**, không phải phát hiện lỗi pixel-level tuyệt đối.
- Cơ chế reload theo loại source hiện hỗ trợ tốt nhất cho: Video Capture Device (Windows/macOS/Linux), Media/VLC Source, Browser Source. Với các loại source khác (NDI, DeckLink, capture card qua plugin...), reload sẽ dùng cách chung (ghi lại settings) và có thể không reset được phần cứng triệt để — log sẽ cảnh báo rõ khi rơi vào trường hợp này.
- Nếu thư viện `obs-websocket-js` tải từ CDN thất bại, trang sẽ tự thử lại qua nhiều CDN/đường dẫn dự phòng khi bấm "Kết nối OBS".
- Không cần đóng/mở lại trình duyệt để đổi cấu hình — chỉ cần mở popup "Cài đặt kết nối", sửa rồi lưu.

---

## Xử lý sự cố thường gặp

| Vấn đề | Nguyên nhân có thể | Cách xử lý |
|---|---|---|
| "Không thể kết nối tới IP:Port" | Sai IP/Port, hoặc WebSocket Server chưa bật trong OBS | Kiểm tra lại Tools → WebSocket Server Settings trong OBS |
| "Sai password OBS WebSocket" | Password nhập sai hoặc OBS có đặt password khác | Kiểm tra lại password trong OBS WebSocket Server Settings |
| "Không tìm thấy source ... trong OBS" | Tên source gõ sai hoặc source đã bị đổi tên/xóa | Sửa lại đúng tên source (phân biệt hoa/thường) |
| "Không tải được thư viện obs-websocket-js" | Mạng chặn CDN hoặc mất kết nối Internet | Kiểm tra Internet, thử tải lại trang, hoặc đổi mạng |
| Reload không có tác dụng với source đang dùng | Loại source chưa được hỗ trợ field thiết bị cụ thể | Xem log để biết loại source, có thể cần bổ sung field riêng cho loại đó |
