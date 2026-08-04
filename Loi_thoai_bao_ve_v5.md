# LỜI THOẠI BẢO VỆ KHÓA LUẬN — V5 (26 SLIDES)

**Đề tài:** THỬ NGHIỆM GOOGLE CHRONICLE: ỨNG DỤNG AI TRONG GIÁM SÁT VÀ PHẢN ỨNG AN NINH TRÊN NỀN TẢNG GOOGLE CLOUD

**Tổng:** 16 slide chính + 1 DEMO + 9 backup | ~12 phút trình bày + ~3 phút demo = ~15 phút

## Phân chia người trình bày

| Phần | Slides | Người | Thời gian |
|---|---|---|---|
| Mở đầu + Lý thuyết + Kiến trúc | 1–7 | **NGƯỜI A** | ~4.5 phút |
| *Chuyển giao* | | A → B | ~15 giây |
| AI + Remediation | 8–9 | **NGƯỜI B** | ~2 phút |
| 🎬 DEMO | — | **B demo, A điều khiển slide** | ~3 phút |
| Kết quả + Kết luận | 10–16 | **NGƯỜI B** | ~4.5 phút |

---

## PHẦN CHÍNH (16 slides)

---

### Slide 1 — Trang bìa
👤 NGƯỜI A

🎤 *"Kính chào quý Thầy Cô trong Hội đồng. Em là Lê Tuấn Anh cùng Nguyễn Minh Huy, xin trình bày khóa luận tốt nghiệp: Thử nghiệm Google Chronicle — Ứng dụng AI trong giám sát và phản ứng an ninh trên nền tảng Google Cloud. Bài trình bày gồm 5 phần: Bối cảnh & Mục tiêu, Cơ sở lý thuyết, Thiết kế & Triển khai, Kết quả & Đánh giá, Kết luận & Hướng phát triển."*

⏱️ ~30 giây

---

### Slide 2 — Bối cảnh & Vấn đề
👤 NGƯỜI A

🎤 *"Theo ISC2 năm 2025, toàn cầu thiếu 4.8 triệu chuyên gia an ninh. IBM cho thấy trung bình mất 181 ngày mới phát hiện sự cố, và mỗi vụ rò rỉ thiệt hại gần 5 triệu đô. SOC truyền thống đang đối mặt 3 thách thức lớn: alert fatigue khiến chuyên viên phân tích bỏ sót mối đe dọa thật, thời gian phản ứng kéo dài hàng giờ đến hàng ngày, và chi phí giấy phép SIEM truyền thống rất đắt đỏ. Câu hỏi đặt ra: liệu kiến trúc Cloud-Native kết hợp AI có thể giải quyết?"*

⏱️ ~40 giây

---

### Slide 3 — Mục tiêu & Phạm vi
👤 NGƯỜI A

🎤 *"Về mục tiêu lý thuyết, nhóm em nghiên cứu hạn chế của SIEM/SOC truyền thống, tìm hiểu SOAR và kiến trúc Cloud-Native của Google Chronicle bao gồm UDM và YARA-L. Về thực tiễn, em xây dựng Lab trên GCP, giả lập tấn công đánh cắp thông tin xác thực và rút trộm dữ liệu (credential theft & data exfiltration), thiết kế phát hiện theo sự kiện (event-driven), tích hợp làm giàu ngữ cảnh 4 lớp (context enrichment) và AI kép Gemini/OpenAI, đồng thời thử nghiệm chuyển đổi YARA-L sang SQL trên BigQuery. Phạm vi đã thực hiện: 18 lần kiểm chứng trên GCP, tự xây luồng xử lý thay thế Chronicle native. Ngoài phạm vi: chưa triển khai trên mạng doanh nghiệp thực tế, chưa bao quát toàn bộ MITRE ATT&CK, và không sử dụng giao diện Chronicle do giới hạn giấy phép."*

⏱️ ~50 giây

---

### Slide 4 — Nền tảng lý thuyết
👤 NGƯỜI A

🎤 *"SOC truyền thống dựa trên 3 trụ cột: Con người, Quy trình, Công nghệ — nhưng đang đối mặt alert fatigue, thiếu nhân sự và chi phí đắt đỏ. Điều này thúc đẩy sự ra đời của SOAR — giải pháp tự động hóa điều phối và phản ứng. Google tích hợp SIEM và SOAR trong Chronicle với UDM chuẩn hóa log và YARA-L làm luật phát hiện. Tuy nhiên, do không có giấy phép Chronicle Enterprise, nhóm tự xây luồng xử lý mô phỏng trên GCP. Về AI, nhóm sử dụng Gemini phân tích log với lược đồ JSON cố định và cơ chế con người kiểm duyệt (Human-in-the-loop) để kiểm soát rủi ro ảo giác AI (hallucination)."*

⏱️ ~45 giây

---

### Slide 5 — Mô hình kiến trúc 3 vùng ⭐
👤 NGƯỜI A

🎤 *"Kiến trúc chia 3 vùng. Vùng đỏ — Vùng tấn công — giả lập tin tặc sử dụng gcloud CLI, gsutil và Python SDK với Service Account Key bị đánh cắp. Vùng xanh dương — Vùng nạn nhân — chứa Honeypot bucket với 55 file giả lập dữ liệu nhạy cảm. Vùng xanh lá — Vùng SOC — là phần nhóm tự thiết kế, gồm toàn bộ luồng xử lý event-driven từ phát hiện, làm giàu ngữ cảnh, AI phân tích đến phản ứng tự động. Toàn bộ hạ tầng được quản lý bằng 7 modules Terraform, có thể tái tạo hoàn toàn từ mã nguồn."*

⏱️ ~45 giây

> 💡 Khi hội đồng hỏi kiến trúc → luôn quay lại slide này.

---

### Slide 6 — Luồng phòng thủ Event-Driven (5 bước)
👤 NGƯỜI A

🎤 *"Luồng phòng thủ gồm 5 bước event-driven. Bước 1: Cloud Monitoring phát hiện bất thường khi số lượng tải file vượt ngưỡng 25 lần trong 60 giây. Bước 2: Cloud Function thực hiện làm giàu ngữ cảnh 4 lớp. Bước 3: Gemini AI phân tích mức độ nghiêm trọng và tạo báo cáo. Bước 4: Gửi cảnh báo qua Telegram cho SOC admin phê duyệt. Bước 5: Khi được phê duyệt, hệ thống tự động vô hiệu hóa Service Account và tạo SCC Finding."*

⏱️ ~40 giây

---

### Slide 7 — Detection & Context Enrichment
👤 NGƯỜI A (slide cuối)

🎤 *"Cơ chế phát hiện dùng event-driven: Log-based Metric đếm trên 25 lần objects.get trong 60 giây thì kích hoạt Cloud Function. Function thực hiện làm giàu ngữ cảnh 4 lớp: : IP, vị trí địa lý, công cụ sử dụng, thời gian. Mỗi lớp giúp bổ sung ngữ cảnh để AI đánh giá chính xác hơn. Ví dụ: cùng hành vi tải file, nếu IP nước ngoài kết hợp Python SDK và ngoài giờ hành chính → CRITICAL; ngược lại IP Việt Nam trong giờ → chỉ HIGH."*

📌 **CÂU CHUYỂN GIAO:**
🎤 *"Vừa rồi em đã trình bày kiến trúc tổng thể và cơ chế phát hiện. Bây giờ bạn Nguyễn Minh Huy sẽ trình bày chi tiết về luồng phân loại bằng AI, cơ chế phản ứng tự động và kết quả đánh giá."*

⏱️ ~50 giây (bao gồm chuyển giao)

---

### Slide 8 — AI Triage Pipeline
👤 NGƯỜI B (slide đầu tiên)

🎤 *"Em là Nguyễn Minh Huy. Về luồng phân loại bằng AI: hệ thống sử dụng Gemini 2.5 Flash làm mô hình chính, dự phòng sang GPT-5.4 Mini khi Gemini gặp lỗi. Câu lệnh được thiết kế cố định với lược đồ JSON bắt buộc gồm 7 trường: severity, confidence, should_escalate, summary, reason, recommended_remediation, và service_account_email. Cách tiếp cận này kiểm soát được đầu ra của AI, tránh ảo giác (hallucination). AI chỉ chiếm khoảng 9 giây xử lý — tức 3% tổng thời gian phản ứng."*

⏱️ ~40 giây

---

### Slide 9 — Human-in-the-Loop & Remediation
👤 NGƯỜI B

🎤 *"SOC admin phê duyệt hành động qua Telegram. Link phê duyệt được ký bằng HMAC-SHA256, hết hạn sau 1 giờ và chỉ dùng được 1 lần — đảm bảo an toàn. Khi SOC admin nhấn phê duyệt, hệ thống thực hiện 3 hành động: Bước 1 — vô hiệu hóa Service Account bị xâm phạm qua IAM API; Bước 2 — tạo SCC Finding với ánh xạ theo khung MITRE ATT&CK; Bước 3 — ghi Audit Log vào Cloud Logging làm bằng chứng điều tra hậu sự cố. Toàn bộ luồng phản hồi trong khoảng 10 giây sau khi được phê duyệt."*

⏱️ ~40 giây

---

### 🎬 DEMO (~3 phút) — Slide 10
👤 NGƯỜI B demo, NGƯỜI A điều khiển slide

🎤 *"Bây giờ em sẽ demo trực tiếp toàn bộ pipeline."*

**Kịch bản:**
1. Chạy `attack_simulation.py` → 55 file được tải
2. Chờ Cloud Monitoring trigger (~4 phút) — có thể dùng video quay sẵn phần chờ
3. Xem cảnh báo Telegram → nhấn Approve Remediation
4. Kiểm tra: Service Account bị disabled + SCC Finding được tạo

🎤 *"Như Hội đồng thấy, từ lúc tấn công đến khi nhận cảnh báo mất khoảng 4 phút, và sau khi phê duyệt, Service Account bị khóa trong 10 giây."*

⏱️ ~3 phút

---

### Slide 11 — Sơ đồ luồng End-to-End ⭐
👤 NGƯỜI B

🎤 *"Sơ đồ toàn trình cho thấy toàn bộ thời gian phát hiện — MTTD — khoảng 265 giây, tức 4.4 phút. Phân tích nút thắt cổ chai cho thấy Cloud Monitoring chiếm 253 giây, tương đương 96% tổng thời gian — đây là do metric window 60 giây mặc định. Phần làm giàu ngữ cảnh, phân loại AI và gửi thông báo Telegram chỉ tốn 12 giây. MTTR — thời gian phản ứng sau khi Admin phê duyệt — khoảng 10 giây. Điều này cho thấy nút thắt không nằm ở luồng xử lý của nhóm, mà nằm ở cơ chế gom metric của Cloud Monitoring."*

⏱️ ~45 giây

---

### Slide 12 — Ma trận thử nghiệm
👤 NGƯỜI B

🎤 *"Ma trận gồm 18 lần thử nghiệm — tất cả có cùng hành vi cốt lõi là rút trộm dữ liệu từ Honeypot Bucket, nhưng thay đổi 3 biến số: công cụ tấn công (gsutil và Python SDK), vị trí IP (Việt Nam và nước ngoài), khung giờ (trong giờ và ngoài giờ hành chính). Mục đích: cô lập biến số để đo lường ảnh hưởng của Context Enrichment lên đánh giá AI. Tỷ lệ phát hiện đạt 100% — tất cả các trường hợp thử nghiệm đều được phát hiện và cảnh báo đúng. Đặc biệt, với IP nước ngoài, AI đánh giá CRITICAL 83% trường hợp — cho thấy làm giàu ngữ cảnh giúp AI phân biệt mức độ nguy hiểm hiệu quả hơn."*

⏱️ ~35 giây

---

### Slide 13 — Đánh giá hiệu năng
👤 NGƯỜI B

🎤 *"Tổng hợp hiệu năng: MTTD 4.4 phút so với tiêu chuẩn ngành theo SANS 2024 là 24 đến 48 giờ — nhanh hơn đáng kể. MTTR 10 giây so với tiêu chuẩn là hàng giờ đến hàng ngày. Tỷ lệ phát hiện đạt 100%. Phần lớn chi phí thử nghiệm đều nằm trong Free Tier của GCP. Lưu ý phạm vi so sánh: hệ thống hiện tại chỉ bao quát 1 kịch bản rút trộm dữ liệu (data exfiltration), chưa phải SOC toàn diện."*

⏱️ ~35 giây

---

### Slide 14 — Kết luận
👤 NGƯỜI B

🎤 *"Về lý thuyết, đề tài đã nghiên cứu có hệ thống từ SOC, SIEM, SOAR đến Chronicle, UDM và YARA-L — và ứng dụng trực tiếp vào thiết kế luồng SOAR serverless thay thế khi không có giấy phép Chronicle native. Về thực tiễn, qua 18 kịch bản kiểm chứng: MTTD đạt 4.4 phút — cải thiện đáng kể so với tiêu chuẩn ngành nhờ kiến trúc event-drivenn kết hợp AI trong kịch bản rút trộm dữ liệu (data exfiltration); MTTR khoảng 10 giây sau phê duyệt — tự động qua IAM API và SCC API; làm giàu ngữ cảnh 4 lớp giúp AI phân biệt mức độ rủi ro theo tổng thể ngữ cảnh thay vì chỉ dựa vào hành vi đơn lẻ; và 3 luật YARA-L kiểm chứng trên BigQuery cho kết quả khớp chính xác. Kết luận cốt lõi: đề tài khẳng định tính khả thi của Cloud-Native kết hợp AI trong SOC tự động, phù hợp tổ chức vừa và nhỏ nhờ chi phí tối ưu với mô hình serverless và trả phí theo lượt sử dụng, toàn bộ hạ tầng IaC bằng Terraform có thể tái tạo hoàn toàn từ mã nguồn."*

⏱️ ~45 giây

---

### Slide 15 — Hạn chế & Hướng phát triển
👤 NGƯỜI B

🎤 *"Về hạn chế, thẳng thắn ghi nhận: không có giấy phép Chronicle nên phải tự xây luồng xử lý thay thế; chỉ bao quát 1 kịch bản rút trộm dữ liệu (data exfiltration); nút thắt 4 phút do metric window 60 giây chiếm 96%; mô hình ngôn ngữ lớn có tính bất định; và YARA-L chuyển sang SQL dùng cửa sổ cố định (tumbling window) có thể bỏ sót sự kiện ở biên. Về hướng phát triển: chuyển sang Log Sink thời gian thực để giảm MTTD xuống dưới 30 giây; mở rộng kịch bản leo thang đặc quyền (privilege escalation), di chuyển ngang (lateral movement); triển khai YARA-L trên Chronicle khi có giấy phép với cửa sổ chồng lấn (hop window) khắc phục bỏ sót biên; và tích hợp tình báo mối đe dọa (threat intelligence) như VirusTotal, AbuseIPDB."*

⏱️ ~50 giây

> 💡 Thẳng thắn — hội đồng đánh giá cao sự trung thực về hạn chế.

---

### Slide 16 — Cảm ơn Hội đồng
👤 CẢ 2 NGƯỜI

🎤 *"Chúng em xin cảm ơn quý Thầy Cô trong Hội đồng đã lắng nghe. Kính mời quý Thầy Cô đặt câu hỏi."*

⏱️ ~10 giây

---

## BACKUP — Lời thoại khi bị hỏi

### B1 — YARA-L Rules trên BigQuery
🎤 *"Do không có Chronicle license, em chuyển logic YARA-L sang SQL trên BigQuery. 3 luật: Mass Download đếm trên 25 objects.get trong 60 giây, Off-Hours kiểm tra truy cập ngoài giờ hành chính, và Suspicious Tool phát hiện gsutil hoặc Python SDK. Kết quả query BigQuery cho thấy hệ thống phát hiện chính xác 55 file được tải."*

### B2 — Infrastructure as Code (Terraform)
🎤 *"Toàn bộ hạ tầng được quản lý bằng 7 modules  Terraform: iam, network, storage, logging_data, serverless, scc, monitoring. Bất kỳ ai có mã nguồn đều có thể tái tạo hoàn toàn lab trong vài phút."*

### B3 — AI trong SOC
🎤 *"AI đóng 3 vai trò: phân tích log tự động giúp giảm tải cho chuyên viên phân tích cấp 1, tương quan ngữ cảnh phát hiện mối đe dọa, và tóm tắt sự cố rút ngắn điều tra. Gemini được tích hợp trực tiếp vào giao diện của Google SecOps, cho phép hỏi bằng ngôn ngữ tự nhiên và trả về truy vấn UDM, tự động tóm tắt cảnh báo và đề xuất hành động. Rủi ro ảo giác AI (hallucination ) được kiểm soát bằng lược đồ JSON cố định, xác thực đầu ra, và cơ chế con người kiểm duyệt (human-in-the-loop)."*

### B4 — Ảnh hưởng Context Enrichment
🎤 *"Bảng này cho thấy cùng hành vi gsutil tải file, khi bổ sung IP nước ngoài + ngoài giờ, AI nhận diện CRITICAL với confidence 1.0. Enrichment tạo ra sự khác biệt quyết định trong đánh giá severity."*

### B5 — Dual-AI Fallback
🎤 *"Đây là log thực tế: Gemini trả lỗi HTTP 400, hệ thống tự động chuyển sang GPT-5.4 Mini và xử lý thành công trong cùng request. Tin nhắn Telegram bên phải là do GPT sinh ra — vẫn đầy đủ 7 trường JSON."*

### B6 — Chi phí vận hành
🎤 *"Toàn bộ luồng xử lý dùng dịch vụ serverless nằm trong Free Tier của GCP. Chi phí duy nhất phát sinh là API calls cho mô hình ngôn ngữ lớn, và cả Gemini lẫn GPT đều có free tier giới hạn theo số lượt mỗi phút và mỗi ngày. Ưu điểm serverless: chỉ tính phí khi có sự kiện — không phát sinh chi phí khi hệ thống nhàn rỗi."*

### B7 — Mapping Lý thuyết → Thực tiễn
🎤 *"Bảng đối chiếu cho thấy mỗi khái niệm lý thuyết đều có thành phần mô phỏng trong hệ thống. SOC 3 trụ cột → Lab 3 vùng. SIEM → Cloud Audit Log + BigQuery. SOAR → Cloud Functions event-driven. UDM → làm giàu ngữ cảnh 4 lớp. YARA-L → BigQuery SQL. Không có lý thuyết nào bị treo."*

### B8 — Tại sao Event-driven?
🎤 *"Kiến trúc Event-driven phản ứng nhanh hơn quét định kỳ (polling) và không tốn chi phí khi nhàn rỗi. Tuy nhiên, cả hai đều dùng cửa sổ cố định (tumbling window) nên có thể bỏ sót sự kiện ở biên. YARA-L trên Chronicle dùng cửa sổ chồng lấn (hop window) giúp khắc phục nhược điểm này — đây là lý do hướng phát triển đề xuất triển khai YARA-L khi có giấy phép."*

### B9 — Dual-Pipeline (Bulk vs Real-Time)
🎤 *"Dạ có thể giảm MTTD xuống 10-15 giây. Em đã thử nghiệm luồng B dùng Log Sink trực tiếp thay vì Cloud Monitoring. Tuy nhiên, mỗi sự kiện log đều kích hoạt 1 Cloud Function nên chi phí tăng tuyến tính. Em chỉ áp dụng cho tài sản trọng yếu (Crown Jewel) — còn tải hàng loạt vẫn dùng cửa sổ gom chỉ số để cân bằng chi phí."*

### B10 — Tumbling Window vs Hop Window
🎤 *"Giả sử kẻ tấn công thông minh — biết ngưỡng là 25 file trong 60 giây. Chúng tải 15 file ở giây thứ 55, nghỉ 10 giây, rồi tải tiếp 15 file ở giây thứ 65. Với Tumbling Window, 15 file rơi vào cửa sổ 1, 15 file rơi vào cửa sổ 2 — không cửa sổ nào đạt ngưỡng 25. Nhưng với Hop Window, cửa sổ 30-90s bắt được tất cả 30 file — vượt ngưỡng — phát hiện thành công."*
---

## Phân chia Q&A

### Người A trả lời (Kiến trúc & Lý thuyết)
| Câu hỏi | Trả lời tóm tắt |
|---|---|
| "Tại sao serverless?" | Trả phí theo lượt, tự động mở rộng, không cần máy chủ 24/7 |
| "Chronicle vs Splunk?" | Kiến trúc Cloud-native, UDM chuẩn hóa, tích hợp SIEM+SOAR |
| "Pub/Sub nghẽn?" | Hạn chế: chưa cấu hình cơ chế thử lại và hàng chờ chết (dead-letter queue) → tin nhắn mất sẽ không được xử lý. Hướng phát triển: bổ sung thử lại + hàng chờ chết |
| "SOAR khác SIEM?" | SIEM thu thập & phân tích, SOAR tự động hóa phản ứng |
| "Kẻ tấn công biết honeypot?" | Phòng thủ nhiều lớp (defense-in-depth), không chỉ dựa vào honeypot |
| "UDM vs làm giàu ngữ cảnh?" | UDM chuẩn hóa log đa nguồn + ngữ cảnh; hệ thống 1 nguồn log nên tập trung làm giàu ngữ cảnh |

### Người B trả lời (AI & Bảo mật & Đánh giá)
| Câu hỏi | Trả lời tóm tắt |
|---|---|
| "Gemini bị ảo giác?" | Lược đồ JSON cố định + xác thực 7 trường + con người kiểm duyệt (Human in the loop) |
| "Tại sao 2 mô hình?" | → Lật backup B5 |
| "AI bị tấn công đối kháng?" | Luồng xử lý cố định, AI chỉ phân tích không quyết định |
| "HMAC đủ an toàn?" | Chuẩn công nghiệp + hết hạn sau 1 giờ + chỉ dùng 1 lần |
| "Làm giàu ngữ cảnh cần thiết?" | → Lật backup B4 |
| "So với SOC thương mại?" | PoC chứng minh tính khả thi, phạm vi 1 kịch bản |
| "Giảm MTTD được không?" | → Lật backup B9 |

---

## Tổng thời gian

| Phần | Slides | Thời gian |
|---|---|---|
| Mở đầu + Bối cảnh | 1–3 | ~2 phút |
| Lý thuyết + Kiến trúc | 4–7 | ~3 phút |
| AI + Remediation | 8–9 | ~1.5 phút |
| 🎬 DEMO | 10 | ~3 phút |
| Kết quả + Đánh giá | 11–13 | ~2 phút |
| Kết luận + Hạn chế | 14–16 | ~2 phút |
| **TỔNG** | **16 slide + demo** | **~13.5 phút** |

> Còn ~1.5 phút buffer cho chuyển giao + trường hợp nói chậm hơn dự kiến.
