# Chuyển project Query fan-out sang tài khoản Claude Team

Cập nhật ngày 26/11/2026, thay bản ngày 31/10/2026. Toàn bộ dữ liệu cần thiết đã nằm trong repo này, trừ API key (cố ý không đưa lên GitHub). Trên tài khoản cũ chưa xóa hay tắt thứ gì.

Mức độ chắc chắn được ghi ở từng bước:
- **[Đã xác minh]**: đã kiểm tra bằng công cụ trên tài khoản cũ.
- **[Chưa xác minh trên Team]**: cách làm đúng trên tài khoản cũ, nhưng chưa thử trên tài khoản Team. Tên menu có thể khác, admin Team có thể phải bật tính năng.

## Cần biết trước

- Theo tài liệu chính thức đọc ngày 31/10/2026 (support.claude.com, bài 9267400 và 9267247, chưa đọc lại): khi tài khoản cá nhân và Team **cùng một email**, có thể chọn "Use your organization account only" rồi "Bring your data with you". Chuyển một chiều, tài khoản cá nhân đóng, gói Pro bị hủy. Chuyển theo: chat, project (hướng dẫn và file), memory. Không chuyển: artifact đã publish, skill tự tạo, connector tự thêm, phiên Claude Code trên cloud kể cả routine. Tài liệu không nói về dữ liệu trong trang artifact và môi trường cloud.
- Vì vậy dù chuyển chính thức hay không, vẫn phải dựng lại skill, routine, môi trường cloud và trang Query Fan-Out Generator từ repo này.
- **Giữ tài khoản cũ cho đến khi tài khoản Team chạy được nút Phân tích trên trang mới.**
- Tài khoản Team do admin quản lý. Có thể admin phải bật: Claude Code trên web, kết nối GitHub, connector Claude Code Remote, chia sẻ artifact. **[Chưa xác minh trên Team]**

## Thứ không có trong repo (phải tự giữ)

| Thứ | Lấy ở đâu |
|---|---|
| Key SerpApi | File `knowledge/claude/serpapi-api-key.txt` trong project files cũ, hoặc trang serpapi.com/manage-api-key. Key thuộc tài khoản SerpApi nên dùng lại nguyên được. **[Đã xác minh: key đang chạy trong môi trường "API Serpapi.com" ngày 26/11/2026]** |
| Key Gemini | File `knowledge/claude/gemini-api-key.txt` trong project files cũ (hiện không phần nào của hệ thống dùng) |
| WP_USER, WP_APP_PASSWORD, SERPAPI_KEY của HLB | Cài đặt môi trường "HLB Agent" trên tài khoản cũ |
| 5 ảnh chụp màn hình đã gửi trong chat ngày 08/10 | Không đưa lên vì có thể lộ thông tin tài khoản SerpApi, và không cần cho việc chạy |

Chỗ đặt key trong repo chỉ có file mẫu `knowledge/claude/serpapi-api-key.example.txt`. `.gitignore` chặn file key thật, đừng xóa dòng đó.

## Các bước trên tài khoản Team, theo thứ tự

### 1. Kết nối GitHub
Đăng nhập tài khoản Team, mở https://claude.ai/connect-github và kết nối tài khoản GitHub `haledcoai`. Nếu được hỏi, cài Claude GitHub App cho repo `haledcoai/query-fanout` (và `haledcoai/hlb-agent` nếu dùng HLB). **[Chưa xác minh trên Team]**

### 2. Quy tắc cá nhân
Settings, mục Profile, dán phần dưới dòng gạch trong `memory/quy-tac-ca-nhan.md` vào ô personal preferences.

### 3. Skill /query-fan-out
Settings, mục Capabilities (hoặc Skills), tải lên `skill/query-fan-out.zip`. Không nhận zip thì tạo skill mới và dán `skill/query-fan-out/SKILL.md`. Bản trong repo là bản đang lưu trên tài khoản cũ ngày 26/11/2026 **[Đã xác minh]**, mới hơn bản trong project files (có quy tắc ghi đúng tên nhóm). Mở một phiên mới để kiểm tra: mặc định 20 truy vấn dự đoán, chạy cả máy tính và điện thoại.

### 4. Project mới gắn repo này
Trong Claude Code trên web, tạo project mới tên "Query fan-out", vào Project settings, mục Repositories (resources), thêm `haledcoai/query-fanout`. Khi phiên Claude mở repo, nó tự đọc `CLAUDE.md` nên nắm được quy tắc và quyết định cũ. **[Chưa xác minh trên Team]**

Sau đó:
- Tải file key SerpApi lên project files với đúng đường dẫn `knowledge/claude/serpapi-api-key.txt` (skill đọc key ở đó khi chạy trong chat).
- Nhắn trong project: "Đọc các file trong memory/ của repo và lưu vào project memory". Bước này giúp các phiên sau trong project nhớ quyết định cũ, kể cả khi không mở repo.

### 5. Môi trường cloud có key SerpApi
Project settings, mục Environment, menu Cloud environment, nút bánh răng cạnh môi trường (hoặc tạo môi trường mới):
- Network: chọn mức giới hạn, cho phép `serpapi.com` và `geo.otterly.ai`, bật package managers và MCP servers. **[Đã xác minh: đúng cấu hình môi trường "API Serpapi.com" đang chạy ngày 26/11/2026]**
- Setup script: một dòng `echo 'KEY_SERPAPI' > ~/.serpapi-key`, thay KEY_SERPAPI bằng key thật. Setup script không đọc được biến môi trường đã lưu, nên key phải ghi thẳng trong script. **[Đã xác minh]**
- Không dùng network secret cho SerpApi, SerpApi từ chối header Bearer (lỗi 401, thử ngày 08/10/2026). **[Đã xác minh]**
- Đặt môi trường này làm mặc định cho project.

### 6. Routine của nút Phân tích
Nhắn Claude trong project mới: "Tạo routine mới, không có lịch, giữ phiên, prompt lấy nguyên văn trong routine/query-fan-out-tu-trang.md". Ghi lại ID mới (dạng `trig_...`). Prompt khớp với routine đang chạy trên tài khoản cũ ngày 26/11/2026 **[Đã xác minh]**.

### 7. Trang Query Fan-Out Generator và Lịch sử
Nhắn Claude: "Đăng lại trang từ artifact/query-fan-out-generator.html thành artifact mới, giữ quyền db (collection runs chỉ admin đọc ghi), mcp (Claude Code Remote, fire_trigger), sample, downloads, user. Thay trig_01GwvKbnDm7jnovRY7rie5dq trong mã bằng ID routine mới. Thay link trang cũ trong prompt routine bằng link trang mới. Rồi nạp 47 tài liệu trong artifact/runs.json vào collection runs, giữ nguyên id."

Lưu ý:
- ID routine cũ có đúng 1 lần trong mã trang. Link trang cũ (`RsdqCpsMdV6uUsgnXw7EVU`) có 2 lần trong prompt routine, phải đổi cả hai, nếu không routine sẽ ghi kết quả vào trang cũ. **[Đã xác minh số lần xuất hiện]**
- `runs.json` lấy lại từ trang ngày 26/11/2026: 47 lần chạy, 2.430 truy vấn, giống bản sao lưu 31/10 (không có lần chạy mới). **[Đã xác minh]**
- Ở lần bấm Phân tích đầu tiên, trang mới sẽ hỏi quyền dùng Claude Code Remote, bấm Cho phép.
- Theo tài liệu ngày 31/10, gói Team không có publish công khai, trang mới chỉ chia sẻ trong tổ chức. Link công khai của trang cũ sẽ ngừng khi tài khoản cũ đóng. **[Chưa xác minh trên Team]**

### 8. Kiểm tra
Bấm Phân tích một truy vấn ngắn trên trang mới, chờ khoảng 5 phút. Kết quả đúng phải có nguồn Google (Autocomplete, Related hoặc PAA). Nếu chỉ có Dự đoán, kiểm tra lại bước 5 (Setup script và network).

### 9. Routine Điều phối HLB và các artifact khác (project HLB)
- Prompt mới nhất trong `routine/dieu-phoi-hlb.md` (ID cũ `trig_015wH6yUw1ZwT6sW57KQANC2`, lịch 8:04 sáng đang tắt, bảng vẫn gọi khi thao tác). Bản 31/10 giữ trong `routine/dieu-phoi-hlb-ban-31-10.md` để tham khảo.
- Cần repo `haledcoai/hlb-agent` (đã có trên GitHub) và môi trường có WP_USER, WP_APP_PASSWORD, SERPAPI_KEY.
- Bảng điều phối HLB và các trang HLB khác chưa được sao lưu mã và dữ liệu. Danh sách và link trong `artifact/danh-sach-artifacts.md`. Claude Doc thì mở link, dùng Export để tải về trước khi đóng tài khoản cũ.

## Những thứ không chuyển được
- Các thread của project cũ và phiên Claude Code trên cloud. Ngữ cảnh quan trọng đã tóm trong `CLAUDE.md` và `memory/`.
- Artifact đã publish và link chia sẻ công khai.
- Lượt dùng SerpApi trong tháng vẫn tính trên tài khoản SerpApi.
