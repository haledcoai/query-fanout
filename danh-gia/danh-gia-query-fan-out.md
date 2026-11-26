# Đánh giá Query Fan-Out Generator

Phạm vi: skill `/query-fan-out` (SKILL.md, 16,5 KB) và trang Query Fan-Out Generator (artifact RsdqCpsMdV6uUsgnXw7EVU). Đánh giá ngày 08/10/2026.

## Kết luận nhanh

Thiết kế tốt ở phần quan trọng nhất: tách bạch dữ liệu thật và dữ liệu mô phỏng (Thật, Google PAA/Related/Autocomplete, Suy ngược, Otterly, Dự đoán), không gọi Otterly hay suy ngược là "thật". Phân 7 nhóm, ý định, loại nội dung đủ để lên kế hoạch nội dung.

Điểm yếu lớn nhất: phần "thật" còn mỏng. Chỉ có Claude, và cũng chưa hẳn là Claude mà người dùng gặp trên claude.ai. Còn lại phần lớn danh sách là mô phỏng (Otterly + 20 đến 50 truy vấn dự đoán), dễ làm loãng tín hiệu thật.

Điểm chấm (chủ quan): thiết kế 8/10, độ tin cậy dữ liệu 5/10, độ ổn định khi chạy 4/10.

## 1. Lỗi cần sửa ngay (đã kiểm tra)

### 1.1 Trang hiển thị dữ liệu Google thành "Dự đoán"
Trong trang, hàm `srcKind` chỉ biết 3 nguồn: Thật, Otterly, Dự đoán. Skill lại ghi thêm nguồn "Google PAA", "Google Related", "Google Autocomplete", "Suy ngược AI Mode", "Suy ngược AI Overview".
Hậu quả khi mở lại một lần chạy từ skill trong Lịch sử:
- Truy vấn PAA, Related, Autocomplete (dữ liệu người dùng Google thật) hiện badge xám "Dự đoán", vì `srcBadge` thay nhãn bằng "Dự đoán" khi không nhận ra nguồn.
- Bộ lọc Nguồn không có Google và Suy ngược. Lọc "Dự đoán" sẽ ra lẫn cả dữ liệu Google.
- Dòng đếm nguồn cộng dữ liệu Google vào "Dự đoán".
Cách sửa: thêm 2 loại nguồn "Google" và "Suy ngược" vào `SOURCES`, `srcKind`, `countBy`, màu badge riêng.

### 1.2 Bộ lọc năm cũ bỏ nhầm truy vấn hợp lệ
Chạy thử regex của trang (cùng quy tắc trong skill) với năm hiện tại 2026:

| Truy vấn | Kết quả | Đúng/sai |
|---|---|---|
| đèn 2000 lumen | giữ | đúng |
| đèn 2000W | giữ | đúng |
| giá 2000đ một bộ | bỏ, "năm cũ 2000" | sai |
| đèn chạy 2000 giờ liên tục | bỏ, "năm cũ 2000" | sai |
| đèn đường QCVN 09:2017, TCVN 7114-1:2008 | bỏ, "năm cũ 2017", "năm cũ 2008" | sai, đây là số hiệu tiêu chuẩn |

Nguyên nhân: `\b` trong JavaScript không coi "đ", "ờ" là chữ, nên "đ" và "giờ" không bao giờ khớp đơn vị. Số hiệu tiêu chuẩn dạng "số:năm" bị coi là năm cũ, trong khi đây là nhóm truy vấn quan trọng với khách B2B ngành chiếu sáng.
Cách sửa: thay `\b` bằng `(?![\p{L}\d])` kèm cờ `u`; không tính là năm nếu ngay trước là dấu hai chấm hoặc gạch nối sau mã TCVN, QCVN, IEC, ISO.

### 1.3 Skill trỏ sai chỗ sau khi nâng cấp project
- Skill đọc key qua "Projects project_read" với đường dẫn `claude/serpapi-api-key.txt`. Sau nâng cấp, file nằm ở `/mnt/project-files/knowledge/claude/`.
- Bước 0a, 1b, 0b cần trình duyệt (Claude_Browser hoặc Claude in Chrome). Phiên cloud của project hiện không có công cụ trình duyệt, và máy chủ cloud bị chặn tới serpapi.com và geo.otterly.ai (đã thử, không kết nối được). Chạy skill ở đây sẽ chỉ còn Bước 1, 2, 3, 4.

## 2. Vấn đề về độ tin cậy dữ liệu

### 2.1 Nhãn "Thật" của Claude chưa hẳn là thật
- 5 subagent chạy trong Claude Code, khác hệ thống và công cụ tìm kiếm của claude.ai mà khách hàng dùng. Prompt cũng nói rõ đây là bài phân tích, có thể làm Claude tìm khác bình thường.
- Subagent tự liệt kê lại các query đã chạy. Không có bước đối chiếu với log công cụ, nên vẫn có khả năng ghi sai hoặc bỏ sót.
- Tần suất 4/5 là độ lặp lại của cùng một model, không phải độ phổ biến trên thị trường.
Đề xuất: đổi nhãn thành "Claude (thật)" và ghi chú môi trường chạy. Khi chạy trong chat claude.ai (không có subagent), 1 lượt đó lại gần với trải nghiệm thật hơn.

### 2.2 Chưa dùng Gemini API dù đã có key
Gemini API khi bật công cụ Google Search trả về trường `groundingMetadata.webSearchQueries`, tức là truy vấn Gemini thực sự gửi lên Google. Đây là nguồn "thật" thứ hai, gần với cách Google AI Mode tách truy vấn hơn mọi mô phỏng.
Đã thử với key trong project: key gọi model bình thường được, nhưng gọi kèm Google Search bị báo hết quota trên 4 model (gemini-2.5-flash đã bị khóa với người dùng mới; gemini-flash-latest, gemini-3.8-flash, gemini-3.1-flash-lite báo vượt quota). Chưa xác định được là gói miễn phí không còn grounding hay hết lượt trong ngày; nhiều khả năng cần bật billing trên Google AI Studio.

### 2.3 Mô phỏng chiếm phần lớn danh sách
Trang mặc định 20, skill mặc định 30 truy vấn dự đoán, cộng 30 đến 50 truy vấn Otterly (cũng do AI tạo). Một lần chạy điển hình có thể chỉ 10 đến 20% là dữ liệu thật.
Đề xuất: giảm dự đoán còn 10 đến 15 và chỉ dùng để lấp nhóm còn trống; trong bảng, cho xem nhanh "chỉ dữ liệu thật".

## 3. Độ ổn định khi chạy
- Một lần chạy đầy đủ cần: subagent, trình duyệt, Gmail connector, ArtifactData, và chờ email Otterly tối đa 10 phút. Thiếu bất kỳ phần nào là phải bỏ bước.
- Otterly qua form email, mỗi phiên phải xác nhận gửi email: chậm, dễ bị captcha hoặc giới hạn lượt.
- SerpApi tốn 4 đến 5 lượt mỗi lần, gói miễn phí 250 lượt/tháng, khoảng 50 truy vấn gốc mỗi tháng.

## 4. Thứ tự cải thiện đề xuất
1. Sửa trang: nhận đủ nguồn Google và Suy ngược (mục 1.1). Lỗi này đang làm sai lệch kết quả đã lưu.
2. Sửa bộ lọc năm (mục 1.2) ở cả trang và skill.
3. Thêm Gemini grounding làm nguồn thật thứ hai, sau khi bật billing hoặc xác nhận còn quota (mục 2.2). Gọi được trực tiếp từ máy chủ cloud, không cần trình duyệt.
4. Cập nhật đường dẫn key và bỏ phụ thuộc trình duyệt cho SerpApi nếu chạy được từ thiết bị của bạn.
5. Giảm số truy vấn dự đoán, thêm chế độ "chỉ dữ liệu thật".
