# CLAUDE.md: project Query fan-out (HALEDCO)

Repo này là bản chuyển của project "Query fan-out" từ tài khoản Claude cá nhân cũ sang tài khoản Claude Team (đẩy lên ngày 26/11/2026). Đọc file này trước khi làm việc. Chi tiết từng quyết định nằm trong `memory/`, cách dựng lại mọi thứ nằm trong `HUONG-DAN-KHOI-PHUC.md`.

## Người dùng và cách làm việc

- HALEDCO (haledco.com), sản xuất và cung cấp đèn LED tại Việt Nam, khách chủ yếu B2B (nhà thầu, chủ đầu tư, nhà xưởng, đại lý). Quy tắc viết đầy đủ trong `memory/quy-tac-ca-nhan.md`.
- Trả lời bằng tiếng Việt, đi thẳng vào vấn đề, phương án tốt nhất trước. Không emoji. Tuyệt đối không dùng em dash, chỉ dùng dấu phẩy, dấu chấm hoặc hyphen.
- Không bịa thông số kỹ thuật. Chỉ nói "đã xác minh" khi đã kiểm tra bằng công cụ.
- Người dùng làm việc qua trang Query Fan-Out Generator, không gọi skill trực tiếp. Điều quan trọng là danh sách truy vấn đầy đủ (kể cả Otterly) và trang chạy không lỗi. Phần phụ (nhận xét cuối, danh sách domain trích dẫn, danh sách Otterly bị loại, thứ tự dòng) không quan trọng, đừng tốn công.

## Hệ thống gồm 4 phần

1. **Skill `/query-fan-out`** (`skill/query-fan-out/SKILL.md`, bản mới nhất lấy từ tài khoản ngày 26/11/2026). Lấy truy vấn Claude thật sự tìm, dữ liệu Google qua SerpApi, truy vấn Otterly, rồi gộp với truy vấn dự đoán. Nguồn ghi: Thật / Google / Suy ngược / Otterly / Dự đoán.
2. **Trang Query Fan-Out Generator** (`artifact/query-fan-out-generator.html`). Nút "Phân tích" gọi `fire_trigger` của connector Claude Code Remote vào routine bên dưới, rồi đọc kết quả từ database của trang (collection `runs`). Chỉ khi không chạy đầy đủ được mới chuyển sang phân tích nhanh (Otterly + dự đoán, không có Google) và nói rõ lý do. Có nút "Làm mới". Lịch sử: `artifact/runs.json` (47 lần chạy, 2.430 truy vấn, lấy lại ngày 26/11/2026).
3. **Routine "Query fan-out từ nút trên trang"** (`routine/query-fan-out-tu-trang.md`). Không có lịch, chỉ chạy khi trang gọi. Chạy trong một phiên cloud mới, không đọc được project files, nên key SerpApi phải nằm trong Setup script của môi trường (`~/.serpapi-key`).
4. **Môi trường cloud**: network cho phép `serpapi.com` và `geo.otterly.ai`; Setup script ghi key vào `~/.serpapi-key`.

## Quyết định đã chốt (không tự đổi)

- SerpApi: chạy cả desktop và mobile, gộp và bỏ trùng. `google` và `google_ai_mode` gọi theo từng thiết bị; Autocomplete chỉ gọi 1 lần. Khoảng 5 lượt cho mỗi truy vấn gốc (đôi khi 6-7), gói Free 250 lượt/tháng. Dưới 8 lượt còn lại thì bỏ bước SerpApi và ghi rõ lý do.
- SerpApi chỉ nhận key qua tham số `api_key` trên URL. Network secret (header Bearer) bị từ chối 401, đừng dùng.
- Truy vấn dự đoán: mặc định 20, lựa chọn 20/30/40/50, đây là mức trần, chỉ lấp các nhóm có 0-1 truy vấn. Người dùng đã từ chối 10/15.
- Trường `nhom` chỉ ghi đúng 1 trong 7 chuỗi: Diễn đạt lại, Liên quan, Ngầm định, Cụ thể hơn, So sánh, Thay thế, Theo ngữ cảnh. Ghi kèm định nghĩa thì trang không hiện được.
- Otterly: người dùng tự gửi truy vấn trên geo.otterly.ai. Trước khi phân tích thì tải CSV Otterly lên trang; sau khi phân tích thì dán link kết quả trong email Otterly vào ô "Thêm Otterly". KHÔNG để phiên chạy tự gửi form lên Otterly bằng email người dùng (đã thử 08/10/2026, bị lớp an toàn chặn và làm mất luôn dữ liệu Google của lượt đó).
- Routine không đăng gì lên chat, chỉ ghi kết quả và lỗi vào database của trang.
- Trước khi chạy `/query-fan-out`, đối chiếu skill đang nạp với các quy tắc trên. Phiên cache skill lúc bắt đầu, muốn đọc bản mới phải mở phiên mới. Nếu skill đang nạp là bản cũ, nói rõ ở đầu kết quả.
- Trước khi khẳng định có dữ liệu Google, kiểm tra `serpapi.com` truy cập được. Không có thì nói thẳng là lượt chạy không có dữ liệu Google.

## Bảo mật

- Không bao giờ commit key hay secret (SerpApi, Gemini, WordPress). `.gitignore` đã chặn các file key. Key thật đặt ở: project files (`knowledge/claude/serpapi-api-key.txt`) và Setup script của môi trường cloud.
- Không in key ra log, chat, file kết quả hay trang.

## Cấu trúc repo

| Đường dẫn | Nội dung |
|---|---|
| `HUONG-DAN-KHOI-PHUC.md` | Các bước dựng lại trên tài khoản Team, ghi rõ chỗ đã và chưa xác minh |
| `skill/` | SKILL.md và file zip để tải lên phần Skills |
| `artifact/` | Mã trang, Lịch sử (`runs.json`), danh sách 17 artifact trên tài khoản cũ |
| `routine/` | Prompt routine của trang và routine Điều phối HLB |
| `memory/` | Bản sao project memory và quy tắc cá nhân |
| `knowledge/` | Hồ sơ năng lực HALEDCO 2026, file mẫu chỗ đặt key |
| `du-lieu/`, `danh-gia/`, `lich-su/` | CSV kết quả mẫu, bản đánh giá skill ngày 08/10, ghi chú nâng cấp project |

## Khi có thay đổi

Khi đổi skill, trang, routine hay ra quyết định mới: cập nhật file tương ứng trong repo và phần "Quyết định đã chốt" ở trên, rồi commit, để phiên sau tự nắm.
