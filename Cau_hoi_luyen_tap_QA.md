# CÂU HỎI LUYỆN TẬP BẢO VỆ KHÓA LUẬN

**Đề tài:** Thử nghiệm Google Chronicle: Ứng dụng AI trong giám sát và phản ứng an ninh trên nền tảng Google Cloud

**Cách sử dụng:** Nhờ 1 người đọc câu hỏi → tập trả lời miệng → so sánh với gợi ý bên dưới.

---

## 1. KIẾN TRÚC & HẠ TẦNG (Người A)

### 1.1 ⭐ "Tại sao chọn kiến trúc serverless thay vì triển khai trên máy chủ truyền thống?"
**Độ khó:** Dễ

> *"Dạ, serverless phù hợp vì 3 lý do: Thứ nhất, trả phí theo lượt sử dụng — khi không có sự cố, hệ thống không tốn chi phí. Thứ hai, tự động mở rộng khi có nhiều cảnh báo đồng thời. Thứ ba, không cần quản trị máy chủ 24/7, phù hợp tổ chức vừa và nhỏ thiếu nhân sự. Toàn bộ thử nghiệm nằm trong Free Tier của GCP, chi phí gần bằng 0."*

---

### 1.2 ⭐ "Tại sao chọn event-driven mà không dùng quét định kỳ (polling)?"
**Độ khó:** Trung bình

> *"Dạ, event-driven phản ứng ngay khi sự kiện xảy ra, không cần chờ chu kỳ quét. Ngoài ra, không tốn chi phí khi hệ thống nhàn rỗi. Tuy nhiên, cả event-driven lẫn polling đều dùng cửa sổ cố định (tumbling window) nên có thể bỏ sót sự kiện ở biên. YARA-L trên Chronicle dùng cửa sổ chồng lấn (hop window) khắc phục nhược điểm này — đây là hướng phát triển em đề xuất."*

---

### 1.3 "Nếu Pub/Sub bị nghẽn hoặc mất tin nhắn thì sao?"
**Độ khó:** Trung bình

> *"Dạ thẳng thắn ghi nhận đây là hạn chế. Hiện tại hệ thống cấu hình chính sách DO_NOT_RETRY cho Pub/Sub — nếu Cloud Function không xử lý được, tin nhắn sẽ bị mất. Chưa có cơ chế thử lại hay hàng chờ chết (dead-letter queue). Hướng phát triển sẽ bổ sung retry policy và dead-letter topic để đảm bảo không mất cảnh báo."*

---

### 1.4 "Tại sao dùng Cloud Monitoring thay vì Log Sink trực tiếp?"
**Độ khó:** Khó

> *"Dạ em đã thử nghiệm cả hai. Log Sink giảm MTTD xuống 10-15 giây, nhưng mỗi sự kiện log đều kích hoạt 1 Cloud Function nên chi phí tăng tuyến tính — không phù hợp kịch bản tải hàng loạt. Cloud Monitoring gom chỉ số trong cửa sổ 60 giây, tuy chậm hơn nhưng tiết kiệm chi phí. Em đề xuất kết hợp: Log Sink cho tài sản trọng yếu, Cloud Monitoring cho phần còn lại. Chi tiết ở slide backup B9."*

---

### 1.5 "Terraform có vai trò gì? Tại sao không cấu hình thủ công trên GCP Console?"
**Độ khó:** Dễ

> *"Dạ, Terraform đảm bảo tính tái lập. Bất kỳ ai có mã nguồn đều tái tạo được toàn bộ hạ tầng trong vài phút. Nếu cấu hình thủ công, rất khó kiểm chứng và dễ sai sót khi triển khai lại. Ngoài ra, 7 modules Terraform giúp quản lý hạ tầng theo từng thành phần: iam, network, storage, logging_data, serverless, scc, monitoring — dễ bảo trì và mở rộng."*

---

### 1.6 "Hệ thống có khả năng mở rộng (scalability) không?"
**Độ khó:** Trung bình

> *"Dạ có ạ. Cloud Functions tự động mở rộng theo số lượng sự kiện — nếu có 10 cảnh báo đồng thời, GCP sẽ tạo 10 instance song song. Pub/Sub có khả năng xử lý hàng triệu tin nhắn mỗi giây. BigQuery mở rộng không giới hạn cho phân tích log. Tuy nhiên, cần lưu ý giới hạn API rate của Gemini và GPT — nếu cảnh báo quá nhiều, có thể bị giới hạn lượt gọi."*

---

## 2. AI & MÔ HÌNH NGÔN NGỮ (Người B)

### 2.1 ⭐ "Gemini có thể bị ảo giác (hallucination) không? Làm sao kiểm soát?"
**Độ khó:** Khó — rất hay bị hỏi

> *"Dạ, mọi mô hình ngôn ngữ lớn đều có rủi ro ảo giác. Em kiểm soát bằng 3 cơ chế: Thứ nhất, lược đồ JSON cố định bắt buộc 7 trường — nếu AI trả thiếu hoặc sai định dạng, hệ thống từ chối và chuyển sang mô hình dự phòng. Thứ hai, xác thực đầu ra — severity phải nằm trong LOW/MEDIUM/HIGH/CRITICAL, confidence từ 0.0 đến 1.0. Thứ ba, cơ chế con người kiểm duyệt — AI chỉ đề xuất, SOC admin quyết định có phê duyệt hay không. AI không tự động khóa tài khoản."*

---

### 2.2 "Tại sao dùng 2 mô hình AI? 1 mô hình không đủ sao?"
**Độ khó:** Trung bình

> *"Dạ, Gemini là mô hình chính nhưng có thể gặp lỗi — ví dụ HTTP 400, quá tải, hoặc bảo trì. Nếu chỉ dùng 1 mô hình, toàn bộ pipeline sẽ ngừng hoạt động. GPT-5.4 Mini làm dự phòng đảm bảo hệ thống luôn hoạt động. Em có log thực tế cho thấy Gemini trả lỗi 400, hệ thống tự động chuyển sang GPT và xử lý thành công trong cùng yêu cầu — chi tiết ở slide backup B5."*

---

### 2.3 "AI có thể bị tấn công đối kháng (adversarial attack) không?"
**Độ khó:** Khó

> *"Dạ, đây là rủi ro thực tế. Tuy nhiên, trong hệ thống này, kẻ tấn công không tương tác trực tiếp với AI — dữ liệu đầu vào của AI đến từ Cloud Audit Logs và kết quả làm giàu ngữ cảnh, không phải từ input của kẻ tấn công. Ngoài ra, luồng xử lý cố định: AI chỉ phân loại, không tự quyết định hành động. Quyết định cuối cùng thuộc về SOC admin qua nút phê duyệt."*

---

### 2.4 "Làm giàu ngữ cảnh (Context Enrichment) có thực sự cần thiết không?"
**Độ khó:** Trung bình

> *"Dạ rất cần thiết. Em có dữ liệu so sánh: cùng hành vi gsutil tải file, không có enrichment thì AI đánh giá MEDIUM với confidence 0.6. Khi bổ sung IP nước ngoài kết hợp ngoài giờ hành chính, AI nhận diện CRITICAL với confidence 1.0. Enrichment giúp AI phân biệt được mức độ rủi ro theo tổng thể ngữ cảnh, thay vì chỉ dựa vào hành vi đơn lẻ. Em có thể mở Telegram để Hội đồng thấy trực tiếp nhiều kịch bản khác nhau."*

---

### 2.5 "Tại sao chọn Gemini 2.5 Flash mà không phải GPT-4 hay Claude?"
**Độ khó:** Trung bình

> *"Dạ, Gemini 2.5 Flash có 3 ưu điểm trong bối cảnh này: Thứ nhất, tích hợp native với GCP — gọi API trực tiếp không cần proxy hay cấu hình phức tạp. Thứ hai, tốc độ phản hồi nhanh — chỉ khoảng 9 giây cho phân tích đầy đủ. Thứ ba, có free tier phù hợp thử nghiệm. GPT được chọn làm dự phòng vì có API ổn định và độc lập với hạ tầng Google — nếu GCP gặp sự cố, GPT vẫn hoạt động."*

---

### 2.6 "Prompt cho AI được thiết kế như thế nào?"
**Độ khó:** Trung bình

> *"Dạ, prompt được thiết kế theo nguyên tắc cố định: cung cấp cho AI toàn bộ ngữ cảnh đã làm giàu — IP, vị trí địa lý, công cụ, thời gian — và yêu cầu trả về JSON với 7 trường bắt buộc. Em không để AI tự do viết văn bản, mà ép nó vào lược đồ cố định. Nếu đầu ra không đúng định dạng, hệ thống coi là lỗi và chuyển sang mô hình dự phòng."*

---

## 3. BẢO MẬT (Người B)

### 3.1 ⭐ "HMAC-SHA256 có đủ an toàn cho link phê duyệt không?"
**Độ khó:** Trung bình

> *"Dạ, HMAC-SHA256 là chuẩn công nghiệp, được dùng rộng rãi trong xác thực API (AWS, Stripe). Link phê duyệt có 3 lớp bảo vệ: chữ ký HMAC chống giả mạo — nếu ai đó sửa URL mà không biết khóa bí mật, chữ ký sẽ không khớp; hết hạn sau 1 giờ; và chỉ dùng được 1 lần — sau khi nhấn, hệ thống đánh dấu đã sử dụng."*

---

### 3.2 "Nếu kẻ tấn công biết đây là honeypot thì sao?"
**Độ khó:** Khó

> *"Dạ, hệ thống không chỉ dựa vào honeypot mà áp dụng phòng thủ nhiều lớp (defense-in-depth). Lớp 1: IAM tối thiểu quyền — giới hạn thiệt hại. Lớp 2: Honeypot — phát hiện sớm. Lớp 3: Cloud Audit Logs — ghi nhận mọi hành vi kể cả khi tránh honeypot. Lớp 4: Ngưỡng cảnh báo — phát hiện bất thường. Lớp 5: AI phân tích ngữ cảnh. Lớp 6: Con người kiểm duyệt. Nếu kẻ tấn công tránh honeypot, Audit Logs vẫn ghi lại hành vi thăm dò — có thể mở rộng luật phát hiện trong hướng phát triển."*

---

### 3.3 "Human-in-the-loop có bị chậm trễ nếu admin không online?"
**Độ khó:** Trung bình

> *"Dạ đúng ạ, đây là đánh đổi có chủ đích. Nếu tự động khóa tài khoản mà không cần phê duyệt, có rủi ro khóa nhầm — gây gián đoạn hoạt động kinh doanh. Em ưu tiên an toàn hơn tốc độ. Tuy nhiên, link phê duyệt có hiệu lực 1 giờ và Telegram có thông báo đẩy trên điện thoại — admin có thể phê duyệt từ bất kỳ đâu. Trong hướng phát triển, có thể bổ sung: nếu severity CRITICAL và confidence trên 0.95, hệ thống tự động khóa mà không cần phê duyệt."*

---

### 3.4 "Service Account Key bị đánh cắp — có cách nào phòng tránh tốt hơn không?"
**Độ khó:** Khó

> *"Dạ, trong thực tế Google khuyến nghị sử dụng Workload Identity Federation thay vì Service Account Key — không cần lưu key dạng file JSON. Tuy nhiên, trong đề tài này, em giả lập kịch bản credential theft — tức kẻ tấn công đã lấy được key — để kiểm chứng khả năng phát hiện và phản ứng của hệ thống. Mục đích là chứng minh dù key bị lộ, hệ thống vẫn phát hiện được hành vi bất thường và khóa tài khoản kịp thời."*

---

## 4. ĐÁNH GIÁ & KẾT QUẢ (Người B)

### 4.1 ⭐ "MTTD 4.4 phút — so với SOC thương mại thì sao?"
**Độ khó:** Trung bình

> *"Dạ, theo tiêu chuẩn ngành SANS 2024, thời gian phát hiện trung bình là 24 đến 48 giờ — MTTD 4.4 phút nhanh hơn đáng kể. Tuy nhiên, em lưu ý phạm vi so sánh: hệ thống hiện tại chỉ bao quát 1 kịch bản rút trộm dữ liệu với luật phát hiện đơn giản. SOC thương mại xử lý hàng trăm loại tấn công khác nhau. Đây là PoC chứng minh tính khả thi, không phải sản phẩm thay thế SOC toàn diện."*

---

### 4.2 "96% thời gian nằm ở Cloud Monitoring — có cách giảm không?"
**Độ khó:** Trung bình

> *"Dạ có ạ. 253 giây trong tổng 265 giây là do Cloud Monitoring gom chỉ số với cửa sổ mặc định 60 giây. Có 2 hướng giảm: Một là chuyển sang Log Sink thời gian thực — giảm xuống 10-15 giây nhưng chi phí tăng. Hai là giảm metric window xuống 30 giây — nhưng có thể gây cảnh báo giả. Em đề xuất phương án kết hợp ở slide backup B9."*

---

### 4.3 "18 kịch bản có đủ để đánh giá hệ thống không?"
**Độ khó:** Khó

> *"Dạ, 18 kịch bản bao phủ tổ hợp của 3 yếu tố chính: 2 công cụ tấn công (gsutil, Python SDK) × 3 vị trí IP (Việt Nam, nước ngoài, VPN) × 3 khung giờ (trong giờ, ngoài giờ, cuối tuần). Đây là ma trận đủ để kiểm chứng rằng enrichment ảnh hưởng đến đánh giá severity của AI. Tuy nhiên, em thẳng thắn ghi nhận: chưa kiểm chứng các kịch bản tấn công khác như leo thang đặc quyền hay di chuyển ngang — đây là hạn chế và hướng phát triển."*

---

### 4.4 "Tỷ lệ cảnh báo giả (false positive) là bao nhiêu?"
**Độ khó:** Khó

> *"Dạ, trong 18 kịch bản kiểm chứng, tỷ lệ phát hiện đúng đạt 100% và không có cảnh báo giả — vì tất cả đều là hành vi tấn công thật trên honeypot bucket. Tuy nhiên, em lưu ý: honeypot bucket được thiết kế sao cho bất kỳ ai truy cập đều đáng ngờ — nên false positive rất thấp theo thiết kế. Trong môi trường thực tế với bucket chứa dữ liệu sản xuất, tỷ lệ cảnh báo giả có thể cao hơn và cần điều chỉnh ngưỡng."*

---

### 4.5 "Chi phí thực tế khi triển khai cho doanh nghiệp là bao nhiêu?"
**Độ khó:** Trung bình

> *"Dạ, trong thử nghiệm, toàn bộ nằm trong Free Tier. Nếu triển khai thực tế, chi phí chính gồm: Cloud Functions tính theo số lượt gọi — khoảng 0.40 USD cho 1 triệu lượt. Gemini API tính theo token. Cloud Monitoring miễn phí cho metric cơ bản. Ước tính cho tổ chức nhỏ với khoảng 100 cảnh báo/tháng, chi phí dưới 10 USD/tháng — rẻ hơn rất nhiều so với giấy phép SIEM truyền thống."*

---

## 5. LÝ THUYẾT (Người A)

### 5.1 "SOAR khác SIEM ở điểm nào?"
**Độ khó:** Dễ

> *"Dạ, SIEM — Security Information and Event Management — tập trung thu thập, chuẩn hóa và phân tích log để phát hiện mối đe dọa. SOAR — Security Orchestration, Automation and Response — tập trung tự động hóa phản ứng: khi phát hiện sự cố, SOAR tự động thực hiện các bước xử lý như khóa tài khoản, cách ly máy, thông báo. Trong hệ thống của em, Cloud Audit Log + BigQuery đóng vai trò SIEM, còn Cloud Functions event-driven đóng vai trò SOAR."*

---

### 5.2 "Chronicle là gì? Tại sao không dùng Chronicle native?"
**Độ khó:** Trung bình

> *"Dạ, Google Chronicle là nền tảng SecOps tích hợp SIEM và SOAR, sử dụng UDM chuẩn hóa log đa nguồn và YARA-L làm ngôn ngữ viết luật phát hiện. Lý do không dùng native: Chronicle Enterprise yêu cầu giấy phép doanh nghiệp — không có bản dùng thử cho sinh viên. Do đó nhóm tự xây pipeline mô phỏng trên GCP, ánh xạ từng thành phần Chronicle sang dịch vụ GCP tương ứng. Bảng đối chiếu chi tiết ở slide backup B7."*

---

### 5.3 "UDM là gì? Hệ thống có dùng UDM không?"
**Độ khó:** Trung bình

> *"Dạ, UDM — Unified Data Model — là lược đồ chuẩn hóa log của Chronicle, giúp thống nhất log từ nhiều nguồn khác nhau về cùng một định dạng. Hệ thống của em chỉ xử lý 1 nguồn log — Cloud Audit Logs — nên không cần chuẩn hóa đa nguồn. Thay vào đó, em tập trung vào làm giàu ngữ cảnh 4 lớp — đóng vai trò tương tự phần ngữ cảnh trong UDM."*

---

### 5.4 "YARA-L là gì? Tại sao chuyển sang SQL?"
**Độ khó:** Trung bình

> *"Dạ, YARA-L là ngôn ngữ viết luật phát hiện của Chronicle — cho phép mô tả pattern tấn công như: nếu có hơn 25 sự kiện objects.get trong 60 giây từ cùng 1 tài khoản thì cảnh báo. Do không có Chronicle, em chuyển logic sang SQL trên BigQuery. 3 luật: Mass Download, Off-Hours, và Suspicious Tool — kết quả BigQuery phát hiện chính xác 55 file được tải. Hạn chế: SQL dùng cửa sổ cố định, có thể bỏ sót sự kiện ở biên — Chronicle dùng cửa sổ chồng lấn khắc phục."*

---

## 6. CÂU HỎI MỞ RỘNG & PHẢN BIỆN

### 6.1 ⭐ "Đóng góp chính của đề tài là gì?"
**Độ khó:** Dễ — nhưng rất quan trọng

> *"Dạ, đóng góp chính gồm 3 điểm: Thứ nhất, chứng minh tính khả thi của việc xây dựng pipeline SOAR serverless trên GCP mà không cần giấy phép Chronicle — phù hợp tổ chức vừa và nhỏ. Thứ hai, chứng minh làm giàu ngữ cảnh 4 lớp giúp AI phân loại chính xác hơn — severity thay đổi từ MEDIUM lên CRITICAL khi có đủ ngữ cảnh. Thứ ba, toàn bộ hạ tầng IaC bằng Terraform có thể tái tạo hoàn toàn từ mã nguồn — phục vụ nghiên cứu và giảng dạy."*

---

### 6.2 "Nếu có thêm thời gian, em sẽ làm gì khác?"
**Độ khó:** Dễ

> *"Dạ, em sẽ tập trung 3 điểm: Một, chuyển sang Log Sink thời gian thực giảm MTTD xuống dưới 30 giây. Hai, mở rộng kịch bản leo thang đặc quyền và di chuyển ngang để hệ thống bao quát hơn. Ba, tích hợp tình báo mối đe dọa như VirusTotal hoặc AbuseIPDB để AI có thêm dữ liệu phân tích — ví dụ kiểm tra IP có nằm trong danh sách đen không."*

---

### 6.3 "Hệ thống có thể triển khai trên AWS hoặc Azure không?"
**Độ khó:** Khó

> *"Dạ, kiến trúc tổng thể có thể chuyển đổi vì dựa trên nguyên lý event-driven chung. Cloud Functions tương đương AWS Lambda hoặc Azure Functions. Pub/Sub tương đương Amazon SNS/SQS hoặc Azure Event Grid. Tuy nhiên, một số thành phần gắn chặt với GCP: Security Command Center, Cloud Monitoring metric, và Gemini API. Nếu chuyển sang AWS, cần thay SCC bằng AWS Security Hub và Gemini bằng Amazon Bedrock."*

---

### 6.4 ⭐ "Tại sao đề tài có tên Chronicle nhưng không dùng Chronicle?"
**Độ khó:** Khó — câu hỏi phản biện thường gặp

> *"Dạ, đề tài ban đầu hướng tới thử nghiệm trực tiếp trên Chronicle. Tuy nhiên, Chronicle Enterprise yêu cầu giấy phép doanh nghiệp mà nhóm không có. Thay vì bỏ đề tài, nhóm chuyển hướng: nghiên cứu lý thuyết Chronicle (UDM, YARA-L) rồi ánh xạ sang pipeline tự xây trên GCP. Mỗi thành phần đều có đối chiếu với Chronicle native — bảng mapping ở slide backup B7. Đây cũng là đóng góp: chứng minh có thể mô phỏng Chronicle bằng dịch vụ GCP riêng lẻ."*

---

### 6.5 "Em đánh giá tính mới (novelty) của đề tài như thế nào?"
**Độ khó:** Khó

> *"Dạ, em không khẳng định đề tài có tính mới đột phá. Các thành phần riêng lẻ — serverless, AI phân tích log, honeypot — đều đã tồn tại. Đóng góp chính nằm ở cách kết hợp: xây dựng pipeline SOAR end-to-end hoàn toàn serverless trên GCP, tích hợp AI kép với cơ chế dự phòng, và chứng minh bằng 18 kịch bản thực nghiệm rằng làm giàu ngữ cảnh cải thiện đáng kể chất lượng phân loại AI. Toàn bộ mã nguồn và hạ tầng IaC có thể tái tạo — phục vụ cộng đồng nghiên cứu."*

---

### 6.6 "Nếu xảy ra nhiều sự cố đồng thời, hệ thống xử lý thế nào?"
**Độ khó:** Trung bình

> *"Dạ, Cloud Functions tự động tạo nhiều instance song song — mỗi cảnh báo được xử lý độc lập. Pub/Sub đảm bảo mỗi tin nhắn được xử lý đúng 1 lần. Tuy nhiên, có 2 nút thắt: API rate limit của Gemini (khoảng 15 lượt/phút ở free tier) và Telegram Bot API (30 tin/giây). Nếu vượt giới hạn, cơ chế dự phòng GPT sẽ xử lý phần Gemini bị từ chối. Đây là giới hạn của free tier — triển khai trả phí sẽ có quota cao hơn."*

---

### 6.7 "Hệ thống có ghi nhận được cuộc tấn công nếu kẻ tấn công chỉ tải vài file thay vì hàng loạt?"
**Độ khó:** Khó

> *"Dạ, hiện tại ngưỡng phát hiện là 25 lần objects.get trong 60 giây. Nếu kẻ tấn công tải chậm — ví dụ 1 file mỗi 5 phút — hệ thống sẽ không kích hoạt cảnh báo. Đây là hạn chế của cách tiếp cận dựa trên ngưỡng. Hướng phát triển: sử dụng AI phân tích hành vi bất thường (anomaly detection) thay vì ngưỡng cố định — ví dụ Service Account này chưa bao giờ truy cập bucket này, thì dù chỉ 1 lần cũng cảnh báo."*

---

### 6.8 "Tại sao chọn Telegram làm kênh thông báo mà không phải Slack hay Email?"
**Độ khó:** Dễ

> *"Dạ, Telegram có 3 ưu điểm: Bot API miễn phí và dễ tích hợp — chỉ cần 1 HTTP request. Thông báo đẩy tức thời trên điện thoại — admin nhận cảnh báo mọi lúc. Hỗ trợ inline keyboard cho nút phê duyệt — SOC admin nhấn Approve ngay trong tin nhắn. Slack cũng phù hợp nhưng cần cấu hình Webhook phức tạp hơn. Email có độ trễ cao hơn và dễ bị bỏ qua."*

---

## MẸO TRẢ LỜI

1. **Luôn bắt đầu bằng "Dạ"** — thể hiện tôn trọng
2. **Nếu không biết → thẳng thắn nói** — *"Dạ, em chưa nghiên cứu sâu phần này, nhưng theo hiểu biết của em..."*
3. **Quay lại slide khi cần** — *"Dạ, em xin phép quay lại slide 5 để minh họa..."*
4. **Ghi nhận hạn chế** — Hội đồng đánh giá cao sự trung thực
5. **Không nói quá** — Không so sánh hệ thống với SOC thương mại ở cùng mức độ
6. **Phân chia rõ A/B** — Ai trả lời câu nào, tránh 2 người giành nhau nói
