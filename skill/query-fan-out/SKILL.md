---
name: "query-fan-out"
description: "Phân tích query fan-out cho một truy vấn gốc: lấy truy vấn Claude thật sự tìm kiếm, dữ liệu Google qua SerpApi (AI Mode, AI Overviews, Related, Autocomplete), truy vấn từ otterly.ai, gộp với truy vấn dự đoán, xuất một file kết quả. Dùng khi gõ /query-fan-out kèm truy vấn gốc."
---

# Query fan-out: một truy vấn gốc, một file kết quả

Người dùng chỉ nhập truy vấn gốc. Không hỏi thêm, không chia theo nền tảng. Kết quả cuối là MỘT danh sách gộp, không trùng lặp, không kèm giải thích từng dòng.

Kết quả của skill này dùng chung định dạng với trang Query Fan-Out Generator (https://claude.ai/artifact/JHh61RYvs7sgKX3HjutNck): cùng 7 nhóm, cùng giá trị Ý định, Loại nội dung, cùng cách ghi Nguồn và cùng bộ lọc truy vấn Otterly.

## Đầu vào
- Truy vấn gốc: lấy từ tin nhắn. Thiếu thì hỏi.
- Số truy vấn dự đoán: mặc định 20. Người dùng có thể đổi (20/30/40/50). Đây là mức trần, không phải số phải đạt.
- Biến thể `/query-fan-out otterly <truy vấn gốc>`: chỉ chạy Bước 0b (lấy kết quả Otterly đã gửi trước đó) rồi gộp vào lần phân tích cũ cùng truy vấn gốc.
- Nếu người dùng gửi kèm link kết quả Otterly hoặc file CSV từ nút Export all prompts: bỏ qua Bước 0a, dùng luôn dữ liệu đó ở Bước 0b.

## Thứ tự chạy
Bước 0a (gửi Otterly) -> Bước 1 (truy vấn thật) -> Bước 1b (SerpApi) -> Bước 0b (nhận và lọc kết quả Otterly) -> Bước 2 -> Bước 3 -> Bước 4.
Gửi Otterly trước vì kết quả Otterly đến qua email sau vài phút, trong lúc đó làm Bước 1 và Bước 1b. Bước 1 (subagent) và Bước 1b (trình duyệt) có thể chạy song song.

## Bước 0a. Gửi truy vấn lên Otterly
Dùng trình duyệt có sẵn trong ứng dụng Claude (tool Claude_Browser), nếu không có thì dùng Claude in Chrome.
1. Mở https://geo.otterly.ai/geo/ai-query-fan-out/ (trang đã được cho phép lâu dài; nếu bị hỏi thì gọi request_access scope "site").
2. Ô "Your query or search prompt": gõ ĐÚNG NGUYÊN VĂN truy vấn gốc, không sửa, không thêm dấu.
3. Giữ nguyên 3 chế độ Google AI Overview, Google AI Mode, ChatGPT. Bấm "Start Fan-Out Analysis".
4. Trang hiện form email: Email = haledco.com@gmail.com, First name = Haledco query fan-out, giữ tích "Store your email". TRƯỚC KHI bấm "Process Fan-Out Analysis", hỏi người dùng một câu xác nhận gửi email lên Otterly (đây là nhập dữ liệu cá nhân vào form, cần đồng ý mỗi phiên). Nếu trường đã tự điền sẵn thì chỉ kiểm tra lại.
5. Thấy dòng "The results for your query will be sent to your email when they are ready" là gửi thành công. Ghi lại giờ gửi.
6. Nếu trang đòi đăng nhập, captcha hoặc báo hết lượt: dừng phần Otterly, ghi rõ lý do, tiếp tục các bước khác.

## Bước 1. Lấy truy vấn thật (Claude web search)
Nếu có công cụ Agent: chạy song song 5 subagent general-purpose, mỗi subagent nhận đúng prompt sau (thay <TRUY VẤN GỐC>):

```
Một người dùng Việt Nam gõ vào trợ lý AI: "<TRUY VẤN GỐC>"

Hãy xử lý yêu cầu này đúng như cách bạn trả lời một người dùng bình thường: tự quyết định cần tìm gì, dùng công cụ WebSearch bao nhiêu lần tùy bạn thấy cần để có câu trả lời tốt. Đừng cố tìm nhiều hay ít hơn mức bình thường.

Khi xong, KHÔNG cần gửi câu trả lời cho người dùng. Chỉ trả về đúng một khối JSON:
{"queries": ["<chuỗi query thứ 1 đúng nguyên văn bạn đã gửi vào WebSearch>", "..."]}
Liệt kê nguyên văn, đúng thứ tự, mọi query bạn đã thực sự gửi vào WebSearch. Không thêm query nào bạn không chạy, không sửa chữ.
```

Nếu không có công cụ Agent: tự chạy 1 lượt, trả lời truy vấn gốc như với người dùng bình thường bằng web search, ghi lại nguyên văn các truy vấn đã tìm. Ghi rõ chỉ có 1 lượt.

Gộp truy vấn trùng (không phân biệt hoa thường, khoảng trắng). Tần suất = số lượt có truy vấn đó / tổng số lượt hợp lệ, ví dụ 4/5. Truy vấn thật KHÔNG qua bộ lọc ở Bước 0b, vì đó là dữ liệu thật.

## Bước 1b. Lấy dữ liệu Google qua SerpApi
API key: lấy theo thứ tự biến môi trường SERPAPI_API_KEY, file `~/.serpapi-key`, file `/mnt/project-files/knowledge/claude/serpapi-api-key.txt` (không có thư mục này thì đọc file project `claude/serpapi-api-key.txt` bằng Projects project_read). Không ghi key vào chat, file kết quả hay trang Query Fan-Out Generator.

Cách A (ưu tiên): gọi thẳng từ máy chủ, không cần trình duyệt.
- Kiểm tra kết nối: chạy `curl -s -m 15 -o /dev/null -w '%{http_code}' https://serpapi.com/`. Ra 200 hoặc 3xx là kết nối được. Ra 000 là bị chặn, chuyển sang Cách B.
- Viết một script Python đọc key như trên (không in key ra màn hình hay log), rồi gọi lần lượt:
  1. `https://serpapi.com/account.json?api_key=<KEY>`: đọc `total_searches_left`. Dưới 8 lượt thì bỏ Bước 1b, ghi rõ lý do.
  2. `https://serpapi.com/search.json` với các bộ tham số sau. AI Mode và Google gọi 2 lần, một lần `device=desktop`, một lần `device=mobile`, vì kết quả hai thiết bị khác nhau:
     - `engine=google_ai_mode, q=<TRUY VẤN GỐC>, gl=vn, hl=vi, device=desktop` (a.json) và `device=mobile` (am.json)
     - `engine=google, q=<TRUY VẤN GỐC>, gl=vn, hl=vi, google_domain=google.com.vn, device=desktop` (g.json) và `device=mobile` (gm.json)
     - `engine=google_autocomplete, q=<TRUY VẤN GỐC>, gl=vn, hl=vi` (c.json), chỉ gọi 1 lần vì Autocomplete không phân biệt thiết bị.
     Mỗi lượt chờ tối đa 60 giây.
  3. Với từng file g.json, gm.json: nếu có `ai_overview.page_token` mà chưa có `text_blocks`, gọi thêm `engine=google_ai_overview, page_token=<token>` (o.json, om.json).
  4. Nếu truy vấn gốc dài hơn 4 từ: gọi thêm Autocomplete với cụm từ khóa chính rút gọn như mục 5 bên dưới.
- Lưu từng kết quả JSON vào thư mục tạm, đặt tên như trên: a.json, am.json (AI Mode), g.json, gm.json (Google), c.json (Autocomplete), o.json, om.json (AI Overview). Sau đó bỏ qua mục 1 đến 5, làm tiếp từ mục 6 (Lấy ra) với các file này.

Cách B: nếu Cách A bị chặn, gọi API trong trình duyệt có sẵn của ứng dụng Claude (Claude_Browser), nếu không có thì dùng Claude in Chrome:
1. Mở `https://serpapi.com/account.json?api_key=<KEY>` (bị hỏi quyền thì gọi request_access scope "site"). Đọc `total_searches_left`. Còn dưới 8 lượt thì bỏ Bước 1b, ghi rõ lý do.
2. Trên trang đó, chạy JavaScript bắt đầu 5 lượt gọi song song, KHÔNG chờ kết quả trong cùng lệnh (tool JavaScript tự dừng sau 45 giây):

```js
const K='<KEY>', Q='<TRUY VẤN GỐC>';
const u=p=>'/search.json?'+new URLSearchParams({...p,api_key:K});
window._r={};
const go=(k,p)=>fetch(u(p)).then(r=>r.json()).then(j=>_r[k]=j).catch(e=>_r[k]={error:String(e)});
go('a',{engine:'google_ai_mode',q:Q,gl:'vn',hl:'vi',device:'desktop'});
go('am',{engine:'google_ai_mode',q:Q,gl:'vn',hl:'vi',device:'mobile'});
go('g',{engine:'google',q:Q,gl:'vn',hl:'vi',google_domain:'google.com.vn',device:'desktop'});
go('gm',{engine:'google',q:Q,gl:'vn',hl:'vi',google_domain:'google.com.vn',device:'mobile'});
go('c',{engine:'google_autocomplete',q:Q,gl:'vn',hl:'vi'});
'started'
```

3. Lệnh tiếp theo: `await new Promise(r=>setTimeout(r,20000)); Object.keys(_r)`. Lặp lại tới khi có đủ a, am, g, gm, c (tối đa 3 lần).
4. Với từng `_r.g` và `_r.gm`: nếu `ai_overview.page_token` có giá trị: gọi thêm `engine=google_ai_overview&page_token=<token>` (cũng chạy kiểu bắt đầu rồi chờ như trên) để lấy nội dung AI Overviews đầy đủ. Nếu `_r.g.ai_overview` đã có sẵn `text_blocks` thì không cần gọi thêm.
5. Nếu truy vấn gốc dài hơn 4 từ: gọi thêm Autocomplete với cụm từ khóa chính rút gọn (2 đến 4 từ đầu mang nghĩa, ví dụ "đèn led nhà xưởng").
6. Lấy ra (đọc từng phần bằng JavaScript, trả về JSON gọn):
   - AI Mode: `_r.a.text_blocks` (đoạn, heading, list) và `_r.a.references` (title, link).
   - AI Overviews: `text_blocks` và `references` của ai_overview.
   - People Also Ask: `_r.g.related_questions[].question`.
   - Related Searches: `_r.g.related_searches[].query`.
   - Autocomplete: `suggestions[].value`.
   Lấy ở cả hai thiết bị (a và am, g và gm), rồi gộp: PAA, Related, Autocomplete bỏ trùng nguyên văn; ý suy ngược trùng nhau chỉ giữ một truy vấn; domain trích dẫn gộp chung. Tên nguồn giữ như dưới đây, không tách theo thiết bị. Ghi vào phần ghi chú số truy vấn Google chỉ có ở điện thoại.
   Mỗi lần chạy tốn khoảng 5 lượt SerpApi, có khi 6 đến 7 (gói miễn phí 250 lượt/tháng, đủ khoảng 35 đến 50 truy vấn gốc).

Xử lý:
- PAA, Related, Autocomplete là truy vấn người dùng thật trên Google: giữ nguyên văn, không qua bộ lọc Otterly. Tên nguồn: "Google PAA", "Google Related", "Google Autocomplete". Bỏ các gợi ý chỉ khác truy vấn gốc ở dấu cách hoặc hoa thường.
- Suy ngược từ AI Mode và AI Overviews: mỗi heading hoặc mục list trong text_blocks ứng với một ý mà AI đã tìm. Với mỗi ý, viết 1 truy vấn tìm kiếm ngắn, kiểu từ khóa, có thể dẫn tới đoạn đó; dựa vào tiêu đề các trang trong references để chọn từ ngữ. Không đưa số liệu trong câu trả lời vào truy vấn, không ghi năm, không dấu ngoặc, không tên thương hiệu trừ khi thương hiệu là chủ đề của mục đó. Tên nguồn: "Suy ngược AI Mode", "Suy ngược AI Overview". Ý có ở cả hai thì ghi đủ hai nguồn.
- Ghi lại danh sách domain được AI Mode và AI Overviews trích dẫn, và trang haledco.com nào được trích (nếu có), để dùng ở Bước 4.
- Lỗi, hết lượt hoặc AI Mode không trả kết quả: ghi rõ, tiếp tục các bước khác.

## Bước 0b. Nhận và lọc kết quả Otterly
1. Tìm email kết quả. Cách tốt nhất: connector Gmail, tìm `from:hello@otterly.ai subject:"GEO Query Fan Out - Completed"` gửi sau giờ ở Bước 0a. Chọn đúng email có câu "for the query <TRUY VẤN GỐC> is complete" (so khớp nguyên văn truy vấn gốc, vì hộp thư có email của các truy vấn khác).
   - Chưa có thì đợi rồi tìm lại, mỗi lần cách khoảng 1 phút, tối đa khoảng 10 phút.
   - Không có connector Gmail: nhờ người dùng dán link kết quả vào chat.
   - Hết thời gian vẫn chưa có: làm tiếp Bước 2-4 không có Otterly, báo người dùng gõ `/query-fan-out otterly <truy vấn gốc>` khi nhận được email.
2. Lấy link dạng `https://geo.otterly.ai/geo/ai-query-fan-out/?result=<ID>`. Chỉ dùng ID từ email của người dùng, không thử ID khác.
3. Mở link trong trình duyệt, rồi chạy JavaScript trên trang để lấy cả 3 chế độ một lần (trang render sẵn HTML, không cần bấm từng tab):

```js
const id = new URL(location.href).searchParams.get('result');
const out = {};
for (const m of ['aimode','aio','chatgpt']) {
  const h = await fetch(`./?result=${id}&mode=${m}`).then(r => r.text());
  const d = new DOMParser().parseFromString(h, 'text/html');
  out[m] = [...d.querySelectorAll('table tbody tr')].map(tr => ({ query: tr.cells[0].textContent.trim(), type: tr.cells[1].textContent.trim() }));
}
out
```

   Nếu người dùng gửi file CSV từ Export all prompts thay vì link: file phân cách bằng dấu chấm phẩy, cột query;type;user_intent;reasoning, gồm cả 3 chế độ nhưng không ghi chế độ nào. Khi đó tên nguồn chỉ ghi "Otterly".
4. Kiểm tra trường QUERY trên trang khớp truy vấn gốc. Không khớp thì bỏ, báo người dùng.
5. Tên nguồn: aimode = "Otterly AI Mode", aio = "Otterly AI Overview", chatgpt = "Otterly ChatGPT". Gộp trùng giữa 3 chế độ; một truy vấn có ở nhiều chế độ thì ghi đủ các nguồn, cách nhau bởi dấu phẩy.
6. Giữ nguyên văn truy vấn Otterly, không sửa chữ.
7. LỌC BỎ, không đưa vào kết quả, mọi truy vấn Otterly có:
   - Dấu ngoặc bất kỳ: ( ) [ ] { } hoặc ngoặc toàn khổ.
   - Năm cũ: số 4 chữ số từ 1990 đến năm trước năm hiện tại (ví dụ năm nay 2026 thì bỏ 2024, 2025). Giữ lại năm hiện tại và năm sau. Không tính là năm nếu ngay sau số là đơn vị đo hoặc tiền: W, kW, kWh, lm, lumen, K, mm, cm, m, V, mA, mAh, đ, VND, đồng, lux, giờ, h (ví dụ "2000 lumen" vẫn giữ). Không tính số nằm trong số thập phân hoặc số có dấu chấm phân cách (ví dụ "2.500 đ").
   - Ghi lại danh sách đã lọc kèm lý do ("có dấu ngoặc" hoặc "năm cũ 2024") để báo ở Bước 4.

## Bước 2. Dự đoán truy vấn của 5 nền tảng
Hình dung lần lượt cách Google AI Mode, Google AI Overviews, ChatGPT, Gemini và Claude tách truy vấn gốc, rồi gộp thành một danh sách N truy vấn, bỏ trùng ý và bỏ các truy vấn đã có ở Bước 1, Bước 1b và Bước 0b.
- Google AI Mode: tách rộng và sâu, nhiều câu hỏi ngầm định, mở rộng thông số, tiêu chuẩn, thương hiệu, theo đối tượng và vị trí.
- Google AI Overviews: ít truy vấn, sát truy vấn gốc, câu ngắn kiểu từ khóa.
- ChatGPT, Gemini, Claude: thường tìm truy vấn gốc trước, sau đó gộp nhiều ý vào một câu tìm kiểu từ khóa.
Ngữ cảnh: thị trường Việt Nam, người tìm là nhà thầu, chủ đầu tư, quản lý nhà xưởng, đại lý hoặc người mua lẻ. Phủ đủ 7 nhóm ở Bước 3, mỗi nhóm ít nhất 1 truy vấn (tính cả truy vấn thật, SerpApi và Otterly), ưu tiên các nhóm Otterly còn thiếu. Không bịa số liệu cụ thể trong truy vấn. Không ghi năm, không dùng dấu ngoặc.

## Bước 3. Phân loại mọi truy vấn (thật, SerpApi, Otterly và dự đoán)
- Nhóm: trường `nhom` CHỈ ghi đúng tên nhóm, một trong 7 chuỗi sau: "Diễn đạt lại", "Liên quan", "Ngầm định", "Cụ thể hơn", "So sánh", "Thay thế", "Theo ngữ cảnh". KHÔNG ghi kèm dấu hai chấm hay phần định nghĩa (sai: "Diễn đạt lại: cùng ý định, cách gọi khác"; đúng: "Diễn đạt lại"). Nếu ghi kèm định nghĩa, trang Query Fan-Out Generator sẽ không hiện được truy vấn. Định nghĩa để chọn nhóm:
  - Diễn đạt lại = cùng ý định, cách gọi khác
  - Liên quan = chủ đề gần kề, người dùng quan tâm tiếp theo
  - Ngầm định = câu hỏi chưa nói ra nhưng cần để quyết định
  - Cụ thể hơn = thông số, tiêu chuẩn, linh kiện, thương hiệu, cách tính
  - So sánh = so với giải pháp, công nghệ, thương hiệu khác
  - Thay thế = sản phẩm hoặc giải pháp khác cho cùng nhu cầu
  - Theo ngữ cảnh = theo đối tượng, công trình, vị trí, quy mô
- Với truy vấn Otterly, dùng TYPE của Otterly làm gợi ý nhưng vẫn tự xét theo định nghĩa trên: reformulation thường là Diễn đạt lại, related là Liên quan, implicit là Ngầm định, comparative là So sánh, entity expansion và personalized thường là Theo ngữ cảnh hoặc Cụ thể hơn.
- Ý định (`y_dinh`, chỉ ghi đúng một chuỗi): Thông tin / Thương mại / Điều hướng / Giao dịch
- Loại nội dung nên có (`loai_noi_dung`, chỉ ghi đúng một chuỗi): Bài blog / Trang sản phẩm / Trang danh mục / FAQ / Bảng so sánh / Video
- Trước khi lưu ở Bước 4, kiểm tra lại: mọi giá trị `nhom`, `y_dinh`, `loai_noi_dung` phải khớp đúng một chuỗi trong danh sách trên.

## Bước 4. Xuất kết quả
1. File CSV (UTF-8 có BOM) trong thư mục outputs, tên `fan-out-<truy-van-khong-dau>.csv`, cột (giống nút Tải CSV trên trang): truy_van_goc, query, nhom, y_dinh, loai_noi_dung, nguon, tan_suat, kieu_otterly.
   - nguon = "Thật" (Bước 1), tên nguồn SerpApi (Bước 1b), tên nguồn Otterly (Bước 0b), hoặc "Dự đoán" (Bước 2). Truy vấn trùng giữa nhiều nguồn: ghi đủ các nguồn, cách nhau bởi dấu phẩy, ví dụ "Thật, Google Autocomplete".
   - tan_suat chỉ điền cho dòng có Thật. kieu_otterly là TYPE gốc của Otterly, chỉ điền cho dòng Otterly.
   - Sắp xếp: dòng Thật trước (tần suất giảm dần), rồi Google PAA / Related / Autocomplete, rồi Suy ngược AI Mode / AI Overview, rồi Otterly (có ở nhiều chế độ trước), rồi dự đoán theo thứ tự 7 nhóm. Trong mỗi nhóm nguồn, dòng có nhiều nguồn hơn xếp trước.
2. Lưu vào Lịch sử của trang Query Fan-Out Generator (https://claude.ai/artifact/JHh61RYvs7sgKX3HjutNck) bằng ArtifactData, collection "runs", một document mới: {root, createdAt (ms), count, lang: "vi", source: "skill", items: [{query, nhom, y_dinh, loai_noi_dung, nguon, tan_suat, kieu_otterly}]}. Bỏ trường rỗng. Với biến thể otterly: đọc document cũ cùng root, cập nhật nó (kèm if_version) thay vì tạo mới.
3. Trong chat: một bảng markdown gom theo nhóm, cột Truy vấn con | Ý định | Loại nội dung | Nguồn. Không thêm cột giải thích. Ghi một dòng số lượng theo nguồn, ví dụ: 6 Thật, 20 Google, 8 Suy ngược, 38 Otterly, 30 Dự đoán. Nếu có truy vấn Otterly bị lọc: thêm một dòng "Đã lược bỏ N truy vấn Otterly có năm cũ hoặc dấu ngoặc", liệt kê ngắn các truy vấn đó kèm lý do.
4. Tối đa 4 câu nhận xét cuối: AI thật sự tìm gì, và trang nào trên haledco.com nên đáp ứng. Dựa trên truy vấn Thật và dữ liệu Google (PAA, Related, Autocomplete); truy vấn có ở nhiều nguồn là tín hiệu mạnh hơn. Một câu về trích dẫn: AI Mode và AI Overviews trích domain nào, haledco.com có được trích không và bằng trang nào.

## Quy tắc
- Không sửa chữ truy vấn thật và truy vấn Otterly. Không ghi truy vấn dự đoán là thật.
- Dữ liệu "Thật" chỉ là của Claude web search. PAA, Related, Autocomplete là truy vấn thật của người dùng Google, ghi đúng tên nguồn. Truy vấn "Suy ngược" là Claude suy ra từ câu trả lời AI Mode và AI Overviews, KHÔNG phải truy vấn fan-out thật của Google; không gọi là "Thật". Truy vấn Otterly là mô phỏng do AI của Otterly tạo, KHÔNG phải dữ liệu thật của Google hay ChatGPT; không gọi là "Thật".
- Ngoài các nguồn Google PAA, Google Related, Google Autocomplete, không nói truy vấn nào là truy vấn thật của Google, ChatGPT hay Gemini.
- Viết tiếng Việt đơn giản, không em dash, không emoji.