# CÂU HỎI MỞ RỘNG & NÂNG CAO — BẢO VỆ KHÓA LUẬN

**Mục đích:** Bổ sung các câu hỏi sâu, hiểm, và góc nhìn mà hội đồng có thể hỏi ngoài phạm vi cơ bản.

**Ký hiệu:** ⭐ = Xác suất bị hỏi cao | 🔴 = Câu hỏi bẫy/hiểm

---

## 1. CÂU HỎI VỀ PHƯƠNG PHÁP NGHIÊN CỨU

### 1.1 🔴 "Phương pháp nghiên cứu của đề tài là gì?"
**Độ khó:** Trung bình — dễ quên chuẩn bị

> *"Dạ, đề tài sử dụng phương pháp nghiên cứu thực nghiệm (experimental research). Cụ thể gồm 3 bước: Thứ nhất, nghiên cứu tài liệu — tổng hợp lý thuyết SOC, SIEM, SOAR, Chronicle từ tài liệu chính thức của Google và các nghiên cứu ISC2, SANS. Thứ hai, thiết kế và triển khai — xây dựng hệ thống trên GCP theo kiến trúc đã đề xuất. Thứ ba, thực nghiệm và đánh giá — chạy 18 lần thử nghiệm với tổ hợp biến kiểm soát (công cụ, IP, thời gian), đo MTTD và MTTR, so sánh với tiêu chuẩn ngành."*

---

### 1.2 "Tại sao chọn 25 file làm ngưỡng cảnh báo? Có cơ sở gì không?"
**Độ khó:** Khó

> *"Dạ, ngưỡng 25 lần objects.get trong 60 giây được xác định qua quan sát thực tế: trong hoạt động bình thường, không có Service Account nào truy cập honeypot bucket — vì bucket này được thiết kế hoàn toàn làm bẫy. Do đó, bất kỳ lượt truy cập nào cũng đáng ngờ. Ngưỡng 25 được chọn để lọc hành vi quét đơn lẻ (1-5 file) và chỉ cảnh báo khi có dấu hiệu rút trộm hàng loạt. Trong thực tế triển khai, ngưỡng này cần điều chỉnh theo từng tổ chức dựa trên baseline hành vi bình thường."*

---

### 1.3 🔴 "Biến kiểm soát và biến phụ thuộc trong thực nghiệm là gì?"
**Độ khó:** Khó — hội đồng hay hỏi về phương pháp

> *"Dạ, biến kiểm soát gồm 3 yếu tố: công cụ tấn công (gsutil hoặc Python SDK), vị trí IP (Việt Nam hoặc nước ngoài), và khung giờ (trong giờ hành chính hoặc ngoài giờ). Biến phụ thuộc là kết quả đầu ra của AI: mức độ nghiêm trọng (severity) và độ tin cậy (confidence). Bằng cách thay đổi từng biến kiểm soát, em chứng minh rằng làm giàu ngữ cảnh ảnh hưởng trực tiếp đến đánh giá của AI."*

---

### 1.4 "Em đảm bảo tính lặp lại (reproducibility) của thực nghiệm như thế nào?"
**Độ khó:** Trung bình

> *"Dạ, toàn bộ hạ tầng được quản lý bằng 7 modules Terraform — bất kỳ ai có mã nguồn đều tái tạo được lab trong vài phút. Script tấn công cũng được viết sẵn. Tuy nhiên, em lưu ý: kết quả AI có tính bất định — cùng đầu vào, Gemini có thể trả severity khác nhau giữa các lần chạy. Em giảm thiểu bằng cách đặt temperature = 0.1, nhưng không loại bỏ hoàn toàn. Đây là hạn chế cố hữu của mô hình ngôn ngữ lớn."*

---

## 2. CÂU HỎI KỸ THUẬT SÂU

### 2.1 ⭐ "Temperature 0.1 có ý nghĩa gì? Tại sao không đặt 0?"
**Độ khó:** Trung bình

> *"Dạ, temperature kiểm soát mức độ ngẫu nhiên trong đầu ra của AI. Temperature 0 cho kết quả gần như cố định nhất, nhưng một số mô hình vẫn có sai số nhỏ do lượng tử hóa (quantization). Em chọn 0.1 thay vì 0 vì: một là đảm bảo AI vẫn có khả năng suy luận linh hoạt khi gặp ngữ cảnh mới — temperature 0 có thể khiến mô hình lặp lại cùng pattern quá cứng nhắc; hai là 0.1 đủ thấp để kết quả ổn định nhưng vẫn có biến thể nhỏ phản ánh sự khác biệt ngữ cảnh."*

---

### 2.2 ⭐ "MITRE ATT&CK mapping trong SCC Finding được thực hiện như thế nào?"
**Độ khó:** Khó — phải biết chi tiết code

> *"Dạ, trong hàm `_write_scc_finding` của webhook, em ánh xạ cố định theo kịch bản: chiến thuật chính là EXFILTRATION với kỹ thuật AUTOMATED_EXFILTRATION — vì kẻ tấn công dùng script tự động tải file hàng loạt. Chiến thuật bổ sung là CREDENTIAL_ACCESS với kỹ thuật STEAL_APPLICATION_ACCESS_TOKEN và VALID_ACCOUNTS — vì kẻ tấn công sử dụng Service Account Key hợp lệ bị đánh cắp. Đây là mapping cố định cho kịch bản data exfiltration, nếu mở rộng sang kịch bản khác thì cần mapping tương ứng."*

---

### 2.3 "Cơ chế one-time-use guard hoạt động như thế nào?"
**Độ khó:** Trung bình

> *"Dạ, khi SOC admin nhấn link phê duyệt lần đầu, webhook kiểm tra Service Account đã bị vô hiệu hóa chưa bằng hàm `_is_sa_already_disabled`. Nếu đã bị vô hiệu hóa rồi — nghĩa là link đã được dùng trước đó — hệ thống trả về thông báo 'đã xử lý' và không thực hiện lại. Tức là em dùng trạng thái Service Account (disabled/enabled) làm cờ one-time-use, thay vì lưu trạng thái riêng trong database. Ưu điểm: không cần database. Nhược điểm: nếu admin enable lại SA rồi nhấn link cũ, link sẽ hoạt động lại — nhưng lúc đó link đã hết hạn (TTL 1 giờ)."*

---

### 2.4 "ip-api.com có đáng tin cậy không? Nếu bị chặn hoặc trả sai thì sao?"
**Độ khó:** Trung bình

> *"Dạ, ip-api.com là dịch vụ miễn phí với giới hạn 45 lượt/phút. Nếu bị chặn hoặc timeout, hàm `_geolocate_ip` trả về fallback là country=unknown, city=unknown, isp=unknown — hệ thống vẫn tiếp tục hoạt động nhưng AI sẽ thiếu ngữ cảnh địa lý. Đây là thiết kế graceful degradation — lỗi enrichment không làm sập pipeline. Trong triển khai thực tế, nên thay bằng MaxMind GeoIP2 database chạy local — không phụ thuộc API bên ngoài và nhanh hơn."*

---

### 2.5 "Tại sao dùng HTTP request gọi Gemini API thay vì dùng SDK chính thức?"
**Độ khó:** Trung bình

> *"Dạ, em chọn HTTP request trực tiếp vì 2 lý do: Thứ nhất, giảm kích thước dependency — Cloud Function có giới hạn kích thước deploy, thêm SDK Google AI sẽ tăng thêm nhiều thư viện phụ thuộc. Thứ hai, kiểm soát hoàn toàn request và response — dễ debug khi gặp lỗi. Nhược điểm là phải tự xử lý retry, error parsing — nhưng trong pipeline này, em đã có cơ chế fallback sang OpenAI nên không cần retry Gemini."*

---

### 2.6 🔴 "Hệ thống có xử lý race condition không? Nếu 2 admin nhấn Approve cùng lúc?"
**Độ khó:** Khó

> *"Dạ, hiện tại hệ thống gửi cảnh báo đến 1 chat ID trên Telegram, nên chỉ có 1 admin nhận. Tuy nhiên, nếu mở rộng cho nhóm admin, có thể xảy ra race condition: 2 người nhấn Approve cùng lúc, cả 2 request đến webhook đồng thời. Cơ chế hiện tại: request đầu tiên vô hiệu hóa SA thành công, request thứ hai kiểm tra thấy SA đã disabled nên trả về 'already processed'. Tuy vậy, có khoảng thời gian rất ngắn giữa 2 request mà cả 2 đều thấy SA còn enabled — đây là hạn chế cần giải quyết bằng distributed lock nếu triển khai production."*

---

## 3. CÂU HỎI SO SÁNH & ĐỐI CHIẾU

### 3.1 ⭐ "So với Google Cloud Armor hoặc Cloud IDS, hệ thống có gì khác?"
**Độ khó:** Khó

> *"Dạ, Cloud Armor và Cloud IDS hoạt động ở tầng mạng — phát hiện DDoS, SQL injection, xâm nhập mạng. Hệ thống của em hoạt động ở tầng ứng dụng và danh tính (identity layer) — phát hiện hành vi bất thường của Service Account trên Cloud Storage dựa vào Audit Logs. Hai hệ thống bổ trợ cho nhau chứ không thay thế: Cloud Armor chặn tấn công từ bên ngoài, còn hệ thống của em phát hiện insider threat khi kẻ tấn công đã có credential hợp lệ."*

---

### 3.2 "So sánh với AWS GuardDuty hoặc Azure Sentinel?"
**Độ khó:** Khó

> *"Dạ, AWS GuardDuty và Azure Sentinel là dịch vụ managed detection — Google có dịch vụ tương đương là Chronicle và SCC Premium. Hệ thống của em khác ở chỗ: đây là pipeline tự xây hoàn toàn bằng dịch vụ cơ bản — không dùng managed detection. Ưu điểm: miễn phí, linh hoạt, hiểu rõ từng thành phần. Nhược điểm: phạm vi hẹp, thiếu threat intelligence tích hợp sẵn, và phải tự bảo trì. Trong thực tế, nên dùng managed detection khi có ngân sách — hệ thống này phù hợp cho tổ chức nhỏ hoặc mục đích học tập."*

---

### 3.3 "Wazuh là giải pháp mã nguồn mở miễn phí — tại sao không dùng?"
**Độ khó:** Trung bình

> *"Dạ, Wazuh là SIEM/XDR mã nguồn mở rất tốt nhưng cần máy chủ riêng để chạy — nghĩa là vẫn phải quản trị hạ tầng 24/7 và trả phí máy chủ. Hệ thống của em hoàn toàn serverless — không cần máy chủ, trả phí theo lượt, tự động mở rộng. Ngoài ra, đề tài tập trung vào kiến trúc Cloud-Native trên GCP và thử nghiệm Chronicle — Wazuh không nằm trong phạm vi nghiên cứu. Tuy nhiên, Wazuh là lựa chọn tốt nếu tổ chức có nhân sự quản trị và muốn giải pháp toàn diện hơn."*

---

## 4. CÂU HỎI VỀ ĐẠO ĐỨC & THỰC TIỄN

### 4.1 "Giả lập tấn công trên cloud có rủi ro pháp lý không?"
**Độ khó:** Trung bình

> *"Dạ, toàn bộ thử nghiệm được thực hiện trên project GCP do nhóm sở hữu — không tấn công hệ thống của bên thứ ba. Honeypot bucket chứa dữ liệu giả do nhóm tạo ra, không chứa dữ liệu nhạy cảm thật. Service Account bị 'tấn công' cũng do nhóm tạo và kiểm soát. Về mặt pháp lý, Google Cloud cho phép penetration testing trên tài khoản của chính mình mà không cần xin phép. Tuy nhiên, em lưu ý: nếu triển khai trong tổ chức thực tế, cần có sự đồng ý bằng văn bản trước khi giả lập tấn công."*

---

### 4.2 "AI phân tích dữ liệu nhạy cảm — có vấn đề bảo mật dữ liệu không?"
**Độ khó:** Khó

> *"Dạ, đây là câu hỏi quan trọng. Trong hệ thống, AI chỉ nhận metadata từ Audit Logs — như tên Service Account, IP, thời gian, phương thức gọi — KHÔNG nhận nội dung file thực tế. Tuy nhiên, khi dùng Gemini API qua Google AI Studio free tier, dữ liệu có thể được Google sử dụng để cải thiện sản phẩm. Trong triển khai production, nên dùng Vertex AI API có cam kết bảo mật dữ liệu, hoặc chạy mô hình on-premise. Với OpenAI, cần kiểm tra Data Processing Agreement."*

---

### 4.3 "Nếu AI đánh giá sai severity — ví dụ HIGH thay vì CRITICAL — hậu quả là gì?"
**Độ khó:** Trung bình

> *"Dạ, trong thiết kế hiện tại, severity không ảnh hưởng đến hành động khắc phục — dù HIGH hay CRITICAL, hệ thống đều gửi cảnh báo và chờ admin phê duyệt. Severity chỉ ảnh hưởng đến: thứ nhất, mức độ ưu tiên mà admin nhìn thấy trên Telegram; thứ hai, mức severity ghi vào SCC Finding. Vì có cơ chế con người kiểm duyệt, admin sẽ tự đánh giá lại dựa trên chi tiết reason và summary trước khi quyết định. Trong hướng phát triển, nếu bỏ human-in-the-loop cho CRITICAL, thì đánh giá sai severity sẽ nghiêm trọng hơn."*

---

## 5. CÂU HỎI VỀ EDGE CASES

### 5.1 ⭐ "Nếu kẻ tấn công tải file chậm — 1 file mỗi 10 phút — hệ thống có phát hiện không?"
**Độ khó:** Khó

> *"Dạ không ạ. Ngưỡng hiện tại là 25 lần trong 60 giây — low-and-slow attack sẽ không kích hoạt cảnh báo. Đây là hạn chế cố hữu của phương pháp dựa trên ngưỡng (threshold-based). Hướng phát triển: Một là dùng anomaly detection — phát hiện bất thường dựa trên baseline, ví dụ SA này chưa bao giờ truy cập bucket này. Hai là triển khai YARA-L trên Chronicle với cửa sổ dài hơn — ví dụ 50 file trong 24 giờ. Ba là kết hợp nhiều tín hiệu: IP lạ + truy cập honeypot dù chỉ 1 file = cảnh báo."*

---

### 5.2 "Nếu Cloud Function bị timeout trước khi hoàn thành pipeline?"
**Độ khó:** Trung bình

> *"Dạ, Cloud Function có timeout mặc định 60 giây cho Gemini API call. Nếu Gemini phản hồi chậm hơn 60 giây, request bị timeout và Python ném exception ReadTimeout. Exception này được bắt bởi try/except ở dòng 530 trong main.py — hệ thống chuyển sang OpenAI fallback với timeout 30 giây. Nếu cả OpenAI cũng timeout, toàn bộ pipeline thất bại và tin nhắn Pub/Sub bị mất vì chính sách DO_NOT_RETRY. Đây là hạn chế cần khắc phục bằng retry policy."*

---

### 5.3 "Nếu Telegram Bot bị chặn hoặc Telegram sập?"
**Độ khó:** Trung bình

> *"Dạ, nếu Telegram không gửi được, hàm `_send_telegram_alert` sẽ ném exception, toàn bộ pipeline thất bại. Hệ thống hiện tại chỉ có 1 kênh thông báo duy nhất — đây là single point of failure. Hướng phát triển: bổ sung kênh thông báo dự phòng như email hoặc Slack, và tách việc gửi thông báo ra khỏi pipeline chính — nếu gửi thất bại, vẫn lưu kết quả phân tích AI vào Cloud Logging để không mất dữ liệu."*

---

### 5.4 🔴 "Kẻ tấn công có thể nhấn link Approve để tự phê duyệt không?"
**Độ khó:** Khó — câu bẫy

> *"Dạ không ạ. Kẻ tấn công không thể tự phê duyệt vì 3 lý do: Thứ nhất, link chỉ được gửi đến Telegram chat cụ thể của SOC admin — kẻ tấn công không nhìn thấy link. Thứ hai, dù bằng cách nào đó có link, kẻ tấn công không thể giả mạo vì link được ký bằng HMAC-SHA256 với khóa bí mật lưu trong biến môi trường của Cloud Function — kẻ tấn công không có khóa này. Thứ ba, link chỉ có hiệu lực 1 giờ và 1 lần dùng."*

---

### 5.5 "Nếu kẻ tấn công xóa Audit Logs trước khi hệ thống phát hiện?"
**Độ khó:** Khó

> *"Dạ, trên GCP, Admin Activity audit logs là bắt buộc và không thể tắt hoặc xóa — ngay cả project owner cũng không thể. Đây là bảo đảm của GCP. Data Access logs (ghi objects.get) có thể bị tắt nếu kẻ tấn công có quyền admin dự án — nhưng trong kịch bản của em, kẻ tấn công chỉ có quyền Storage Object Viewer, không có quyền chỉnh cấu hình logging. Ngoài ra, Cloud Monitoring metric đã gom chỉ số trước khi kẻ tấn công kịp thao tác — nên cảnh báo vẫn được kích hoạt."*

---

## 6. CÂU HỎI VỀ TRÌNH BÀY & BÁO CÁO

### 6.1 "Em có thể chạy demo trực tiếp không?"
**Độ khó:** Dễ — nhưng cần chuẩn bị

> *"Dạ, em đã chuẩn bị video demo quay sẵn toàn bộ pipeline thay vì demo trực tiếp. Lý do: hệ thống có thời gian chờ Cloud Monitoring khoảng 4 phút — nếu demo trực tiếp sẽ có khoảng lặng dài và rủi ro lỗi kỹ thuật tại phòng bảo vệ. Video demo được quay với kịch bản thật trên GCP, có tua nhanh phần chờ và giữ nguyên phần pipeline xử lý. Nếu Hội đồng muốn xem thêm, em có thể mở Telegram để xem tin nhắn cảnh báo thực tế từ nhiều lần thử nghiệm."*

---

### 6.2 🔴 "Nguồn trích dẫn ISC2 2025 và IBM là từ đâu?"
**Độ khó:** Dễ — nhưng phải nhớ chính xác

> *"Dạ, số liệu 4.8 triệu chuyên gia thiếu hụt đến từ báo cáo ISC2 Cybersecurity Workforce Study 2025. Số liệu 181 ngày phát hiện và chi phí gần 5 triệu USD đến từ IBM Cost of a Data Breach Report. Em có trích dẫn đầy đủ trong phần Tài liệu tham khảo của báo cáo."*

---

### 6.3 "Tại sao SANS 2024 mà không dùng báo cáo mới hơn?"
**Độ khó:** Trung bình

> *"Dạ, SANS SOC Survey 2024 là báo cáo mới nhất có số liệu MTTD/MTTR chi tiết tại thời điểm nhóm thực hiện nghiên cứu. Em đã kiểm tra và chưa có bản SANS 2025 tương đương. Ngoài ra, tiêu chuẩn MTTD 24-48 giờ khá ổn định qua các năm — sự khác biệt giữa các báo cáo thường nằm ở phương pháp đo chứ không phải con số."*

---

## 7. CÂU HỎI "NẾU... THÌ..."

### 7.1 "Nếu em là CISO của một công ty nhỏ, em có triển khai hệ thống này không?"
**Độ khó:** Trung bình

> *"Dạ, em sẽ dùng hệ thống này như lớp phòng thủ bổ sung, không phải giải pháp SOC duy nhất. Cụ thể: đặt honeypot bucket trong các project quan trọng, kết nối pipeline phát hiện, và cấu hình thông báo cho đội security. Chi phí gần 0 nên không có rào cản ngân sách. Tuy nhiên, em sẽ bổ sung: retry cho Pub/Sub, kênh thông báo dự phòng, và mở rộng kịch bản phát hiện. Song song đó, nếu có ngân sách, em vẫn khuyến nghị dùng Google SCC Premium hoặc Chronicle để có phạm vi bao phủ toàn diện."*

---

### 7.2 "Nếu Google ngừng cung cấp free tier, hệ thống có còn ý nghĩa không?"
**Độ khó:** Trung bình

> *"Dạ, kiến trúc event-driven và serverless vẫn giữ nguyên giá trị — chi phí chỉ phát sinh khi có sự kiện. Ngay cả khi không có free tier, chi phí vẫn rất thấp vì hệ thống chỉ chạy khi có cảnh báo. Ước tính với 100 cảnh báo/tháng, chi phí dưới 1 USD. So với SIEM truyền thống hàng nghìn USD/năm, đây vẫn là lợi thế lớn. Giá trị cốt lõi của đề tài không nằm ở free tier, mà ở kiến trúc serverless kết hợp AI."*

---

### 7.3 "Nếu có Chronicle license, hệ thống này có còn cần thiết không?"
**Độ khó:** Khó

> *"Dạ, nếu có Chronicle license, nhiều thành phần sẽ được thay thế bằng native: YARA-L chạy trực tiếp thay vì SQL trên BigQuery, UDM chuẩn hóa đa nguồn log thay vì chỉ Audit Logs, và Chronicle SOAR thay vì Cloud Functions tự xây. Tuy nhiên, phần AI triage và làm giàu ngữ cảnh của em có thể bổ trợ — Chronicle native chưa có cơ chế tích hợp AI kép Gemini/GPT với fallback. Ngoài ra, cơ chế honeypot + Telegram notification + HMAC approval có thể tích hợp thêm vào Chronicle workflow."*

---

### 7.4 🔴 "Nếu em làm lại từ đầu, em sẽ thay đổi gì?"
**Độ khó:** Trung bình — câu hỏi đánh giá sự trưởng thành

> *"Dạ, em sẽ thay đổi 3 điều: Thứ nhất, thiết kế Log Sink song song ngay từ đầu thay vì chỉ Cloud Monitoring — để so sánh MTTD của 2 phương pháp. Thứ hai, thêm validation giá trị cho đầu ra AI — hiện tại chỉ kiểm tra sự tồn tại của 7 trường mà chưa kiểm tra giá trị severity có hợp lệ không. Thứ ba, mở rộng ít nhất 2-3 kịch bản tấn công khác ngoài data exfiltration — để kết quả đánh giá có tính tổng quát hơn."*

---

## MẸO XỬ LÝ CÂU HỎI KHÓ

1. **Câu hỏi không biết:** *"Dạ, em chưa nghiên cứu sâu phần này trong phạm vi đề tài. Tuy nhiên, theo hiểu biết của em..."* → trả lời theo logic
2. **Câu hỏi bẫy:** Bình tĩnh, phân tích lại câu hỏi trước khi trả lời. Nhiều câu bẫy có tiền đề sai
3. **Câu hỏi dài:** Xin phép ghi chú lại, trả lời từng phần
4. **Câu hỏi so sánh:** Luôn nêu rõ phạm vi — *"Trong phạm vi PoC của em..."*
5. **Câu hỏi về hạn chế:** Không phòng thủ — ghi nhận thẳng thắn rồi đề xuất hướng phát triển
