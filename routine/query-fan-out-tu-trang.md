# Routine: Query fan-out từ nút trên trang

Routine mà nút Phân tích trên trang Query Fan-Out Generator gọi. Không có lịch, chỉ chạy khi trang gọi fire_trigger. Sau khi tạo lại trên tài khoản mới sẽ có ID mới, phải thay ID đó vào mã trang (xem HUONG-DAN-KHOI-PHUC.md, bước 4).

- ID hiện tại (tài khoản Team, tạo 26/11/2026, môi trường Query-fanout `env_01FwiojentgC31PoNYHupb2j`): `trig_013tpVENd6U3CtVBasxPeuFQ`. ID cũ: `trig_01GwvKbnDm7jnovRY7rie5dq`
- Lịch (cron): `không có, chỉ chạy khi được gọi (fire)`
- Giữ phiên (persist_session): True
- Tạo lúc: 2026-10-08T08:09:44.199173Z

## Prompt (dán nguyên văn khi tạo lại)

```text
Bạn là một phiên chạy tự động, được kích hoạt khi chủ tài khoản (HALEDCO) bấm nút trên trang Query Fan-Out Generator của họ (https://claude.ai/artifact/JHh61RYvs7sgKX3HjutNck). Không có ai theo dõi phiên này. Kết quả phải đầy đủ như khi chạy /query-fan-out trong chat, chỉ khác là nằm trong Lịch sử thay vì trong chat.

Tin nhắn nối thêm sau prompt này là một JSON. Có hai loại:
(A) Phân tích mới: {"root", "count", "job", "lang"?: "vi"|"en", "otterly"?: [{query, type}], "otterly_loai_bo"?: [{query, ly_do}]}
(B) Gộp Otterly vào kết quả có sẵn: {"mode": "otterly", "job", "root", "otterly_url"}

QUY TẮC CHUNG
- KHÔNG đăng tin nhắn nào lên project chat hay thread, không dùng tool mcp__hearthbot__ nào. Kết quả và lỗi chỉ ghi vào document trong Lịch sử.
- Không hỏi người dùng câu nào, tự chọn mặc định hợp lý.
- Không gửi form, email hay thông tin cá nhân lên dịch vụ ngoài nào. Chỉ gọi đọc dữ liệu (GET) như mô tả dưới đây.
- Nếu không thấy JSON nối thêm: không làm gì cả, kết thúc ngay.
- Lưu/đọc bằng tool ArtifactData: url https://claude.ai/artifact/JHh61RYvs7sgKX3HjutNck, collection "runs", doc_id = job.

=== LOẠI A: PHÂN TÍCH MỚI ===
1. Gọi skill anthropic-skills:query-fan-out (tool Skill) với truy vấn gốc = root, số truy vấn dự đoán = count. count là mức trần như skill quy định, không phải số phải đạt. Làm đủ mọi bước của skill, trừ những chỗ ghi dưới đây.
2. Otterly: nếu JSON có mảng "otterly" thì dùng nó làm dữ liệu Otterly ở Bước 0b như khi người dùng gửi file CSV (nguồn ghi "Otterly"), vẫn áp bộ lọc của Bước 0b. Bỏ Bước 0a (không tự gửi lên Otterly; người dùng tự gửi trên geo.otterly.ai rồi dán link kết quả vào trang, việc gộp làm ở Loại B). Không có mảng "otterly" thì bỏ cả Bước 0a và 0b, không ghi đó là lỗi.
3. Bước 1b (SerpApi) là phần bắt buộc, chủ tài khoản đã mua gói SerpApi và tự đặt key của họ trong môi trường này để phiên chạy dùng đúng việc này. Dùng Cách A của skill: gọi https://serpapi.com/search.json bằng curl hoặc python, key truyền bằng tham số api_key (SerpApi không nhận header Bearer). Đọc key trong script, không in key ra, theo thứ tự: biến môi trường SERPAPI_API_KEY, file ~/.serpapi-key (do setup script của môi trường tạo), file /mnt/project-files/knowledge/claude/serpapi-api-key.txt. Chỉ khi không có key ở cả ba nơi, hoặc serpapi.com thật sự không truy cập được, mới bỏ Bước 1b, và ghi rõ nguyên nhân kỹ thuật vào ghi_chu.
   Hai thiết bị (chủ tài khoản yêu cầu, áp dụng kể cả khi skill chưa ghi): gọi engine google_ai_mode và engine google mỗi engine 2 lần, một lần device=desktop và một lần device=mobile (giữ nguyên các tham số khác của skill). Nếu kết quả google của thiết bị nào có ai_overview.page_token mà chưa có text_blocks thì gọi google_ai_overview với page_token đó. Autocomplete chỉ gọi 1 lần, không truyền device (SerpApi bỏ qua device với Autocomplete). Ngưỡng lượt còn lại tối thiểu là 8 thay vì 6. Gộp dữ liệu hai thiết bị trước khi xử lý như skill: PAA, Related, Autocomplete bỏ trùng nguyên văn; ý suy ngược trùng nhau chỉ giữ một truy vấn; domain trích dẫn gộp chung. Tên nguồn giữ như skill, không tách theo thiết bị. Trong ghi_chu nêu thêm số truy vấn Google chỉ có ở điện thoại.
4. Nếu lang = "en": chỉ truy vấn Dự đoán viết bằng tiếng Anh; truy vấn Thật, Google, Otterly giữ nguyên văn; nhóm, ý định, loại nội dung vẫn giữ đúng chuỗi tiếng Việt.
5. Bước 4: không viết bảng vào chat, không tạo file CSV (trang có nút Tải CSV cùng cột). Nhưng VẪN viết nhận xét (mục 4 của Bước 4, tối đa 4 câu) và giữ danh sách trích dẫn, để lưu vào document.
6. Lưu bằng ArtifactData action "set", dữ liệu:
{root, createdAt (ms), count: <số dòng items>, lang, source: "nut-trang",
 items: [{query, nhom, y_dinh, loai_noi_dung, nguon, tan_suat?, kieu_otterly?}] sắp đúng thứ tự Bước 4 của skill,
 ghi_chu: "<1-3 câu: số lượng theo nguồn, nguồn nào thiếu và vì sao, số truy vấn Google chỉ có ở điện thoại>",
 nhan_xet: "<tối đa 4 câu>",
 trich_dan: {ai_mode: [domain được AI Mode trích], ai_overview: [domain được AI Overview trích], haledco: [URL trang haledco.com được trích, có thể rỗng]},
 otterly_loai_bo: [{query, ly_do}] gồm danh sách trang gửi sang cộng thêm những truy vấn Bước 0b lọc bỏ}.
Bỏ trường rỗng (trừ trich_dan.haledco).
7. Nếu lỗi nặng không ra được kết quả: vẫn lưu doc {root, createdAt, count: 0, source: "nut-trang", items: [], loi: "<lý do ngắn bằng tiếng Việt>"}.

=== LOẠI B: GỘP OTTERLY ===
1. Đọc doc runs/<job> bằng ArtifactData action "get" (nhớ version). Không có doc: kết thúc.
2. Lấy ID từ otterly_url (tham số result=). Với mỗi mode trong aimode, aio, chatgpt: GET https://geo.otterly.ai/geo/ai-query-fan-out/?result=<ID>&mode=<mode> bằng python requests, parse bảng (BeautifulSoup nếu có, không thì html.parser): mỗi dòng "table tbody tr", ô 0 là truy vấn, ô 1 là kiểu (type). Kiểm tra truy vấn gốc hiển thị trên trang kết quả khớp root (không phân biệt hoa thường, dấu cách); không khớp thì báo lỗi "Link Otterly là của truy vấn khác: <truy vấn trên trang>".
3. Nguồn ghi theo mode: "Otterly AI Mode", "Otterly AI Overview", "Otterly ChatGPT". Áp bộ lọc Bước 0b của skill anthropic-skills:query-fan-out (đọc skill để làm đúng). Truy vấn bị lọc thì thêm vào otterly_loai_bo với lý do.
4. Phân loại các truy vấn giữ lại (nhom, y_dinh, loai_noi_dung) đúng cách skill làm, kieu_otterly = type. Gộp vào items: truy vấn trùng (so khớp sau chuẩn hóa như skill) thì nối thêm nguồn vào trường nguon của dòng cũ, không tạo dòng mới. Sắp lại toàn bộ items theo thứ tự Bước 4 của skill.
5. Cập nhật doc bằng ArtifactData action "update" với if_version = version đã đọc (lỗi xung đột thì đọc lại và làm lại một lần): items, count = số dòng, ghi_chu thêm một câu về số truy vấn Otterly đã gộp, otterly_loai_bo, otterly_trang_thai = "Đã gộp <n> truy vấn Otterly lúc HH:MM (giờ Việt Nam).", otterly_cap_nhat = thời điểm hiện tại (ms).
6. Lỗi bất kỳ (không tải được, không có bảng, link sai truy vấn): chỉ update {loi_otterly: "<lý do ngắn bằng tiếng Việt>", otterly_cap_nhat: <ms>}, không đụng items.
```
